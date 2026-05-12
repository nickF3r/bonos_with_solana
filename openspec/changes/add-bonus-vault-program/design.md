## Objetivo técnico

Programa Anchor que custodia fondos en una **token account SPL** controlada por un **PDA**, mantiene configuración del vault en una cuenta de estado y un **PDA por usuario** para elegibilidad y límites de claim. Las transferencias usan **CPI** al programa SPL Token.

## Instrucciones

### 1. `initialize_vault`

- **Signers**: admin (payer), opcionalmente rent payer.
- **Efecto**: crea/init `VaultState` PDA; crea o asocia la **bonus vault token account** bajo authority PDA; guarda `admin`, `mint`, bumps.
- **Precondición**: mint SPL válido; admin firma.

### 2. `register_user`

- **Signers**: admin.
- **Args**: `user: Pubkey`, `amount: u64`.
- **Efecto**: init o actualiza `UserBonus` PDA para `(user)`; establece `amount_allocated`. **Política fase base**: si el PDA no existe, `init`; si existe, el admin puede **reemplazar** el cupo (`amount_allocated = amount` recibido) para simplificar soporte y tests.
- **Validación**: solo `admin == vault.admin`.

### 3. `claim`

- **Signers**: usuario (dueño del token account destino).
- **Efecto**: calcula `pending = allocated - claimed`; transfiere `min(pending, requested)` o todo el pendiente vía **spl_token::transfer** CPI desde bonus vault ATA → user ATA; actualiza `claimed`.
- **Validación**: `UserBonus.user == signer`; cuentas de token correctas y mint coincide con vault.

## PDAs y seeds (explícitos)

Documento de referencia para clientes y tests; ajustar strings exactos en código a una sola fuente de verdad (`#[constant]` o módulo `seeds`).

| Cuenta | Seeds sugeridos |
|--------|------------------|
| Vault state | `["vault", mint.as_ref()]` o `["vault", program_id.as_ref(), mint.as_ref()]` si se desea vault único por mint |
| Vault authority (signer PDA para token vault) | `["vault_authority", vault_state.key().as_ref()]` o embebido en el mismo PDA que el state si se unifica diseño |
| User bonus | `["user_bonus", vault_state.key().as_ref(), user.as_ref()]` |

**Nota**: Si `VaultState` y authority comparten un solo PDA, simplifica derivación pero mezcla roles; lo habitual es **state account** (datos) + **authority PDA** (solo firma) para la token vault.

## SPL Token y CPI

- **Depósito inicial de fondos al vault**: puede ser instrucción explícita `deposit` (admin o cualquiera) o flujo off-chain con wallet estándar + cliente; mínimo viable: documentar transferencia SPL estándar al ATA del vault desde wallet admin en localnet.
- **Claim**: CPI `Transfer` desde vault token account (owner = authority PDA) con **invoke_signed** usando seeds del authority PDA.

## Stack y entorno

| Pieza | Elección |
|-------|----------|
| Framework programa | Anchor (Rust) |
| Red | Localnet: `solana-test-validator` |
| RPC frontend | `http://localhost:8899` |
| Wallet | Backpack (`@solana/wallet-adapter-wallets` o equivalente Backpack en el stack adapter) |
| Token | SPL Token (clásico), `anchor-spl` |

## Frontend (mínimo)

- `ConnectionProvider` apuntando a localhost8899.
- `WalletProvider` con adapter Backpack + modal estándar.
- Pantallas o acciones: **Admin**: inicializar vault, registrar usuario (inputs pubkey + amount). **Usuario**: claim (monto opcional o máximo).

## Seguridad (base)

- Comprobar **owner** y **mint** de todas las token accounts en cada instrucción.
- No permitir claim que exceda `allocated - claimed`.
- Overflow-safe arithmetic (`checked_add` / `checked_sub`).
- Re-inicialización: usar constraints `init`/`seeds`/`bump` de Anchor para evitar duplicados no deseados.

## Dependencias Rust sugeridas

- `anchor-lang`, `anchor-spl` (feature `token`).
