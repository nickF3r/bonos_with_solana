## Why

El repositorio necesita un programa on-chain base que custodie un pool de tokens SPL, permita al administrador registrar montos de bonus por usuario y permita a cada usuario reclamar (retirar) solo lo asignado. Sin este programa no hay fuente de verdad on-chain para elegibilidad ni límites de claim.

## What Changes

- **Programa Anchor** (`programs/`): cuenta de estado del vault, PDAs por usuario para registro de bonus, e instrucciones `initialize_vault`, `register_user` (admin) y `claim` (usuario), usando SPL Token para depósitos y retiros.
- **Cliente / frontend** (nuevo o bajo `app/`): conexión a **localnet** en `http://localhost:8899`, **Backpack** vía wallet-adapter, flujos para admin (init + registro) y usuario (claim).
- **Tests**: Anchor + flujo local con `solana-test-validator` (o test validator embebido) y cuentas de prueba.

## Impact

| Ámbito | Afectado |
|--------|----------|
| On-chain | Nuevo programa; cuentas PDAs; CPI a SPL Token |
| Off-chain | App TS, scripts de deploy a localnet, documentación de seeds |

### Cuentas de Solana involucradas

- **Mint SPL** del token de bonus (existente o creada en setup local).
- **Vault authority PDA**: firma derivada del programa para la **token account** que custodia el pool (ATA del PDA o cuenta de token asociada al PDA según diseño).
- **Bonus vault (token account)**: custodia los fondos del programa.
- **Vault state PDA**: estado global (admin, mint, bump, contadores si aplica).
- **User bonus PDA** (por usuario): almacena `user` (pubkey), `amount_allocated`, `amount_claimed` (o solo pendiente), `bump`.
- **User token ATA**: destino del usuario al hacer claim.
- **Admin**: firma en `initialize_vault` y `register_user`.

## Alcance explícito (fase base)

- Un solo tipo de bonus en un mint concreto por instancia de vault.
- Registro de usuarios solo por admin on-chain.
- Claim hasta agotar asignación; sin caducidad ni vesting en esta fase salvo que se añada en diseño posterior.

## Fuera de alcance (posterior)

- Merkle trees o airdrop masivo off-chain.
- Multivault o varios mints por programa sin migración.
- Mainnet/devnet deploy (solo localnet en esta propuesta operativa).
