## On-chain (Anchor)

1. Inicializar workspace Anchor en el repo (`Anchor.toml`, `programs/bonus_vault/`, `Cargo.toml` workspace) si aún no existe.
2. Definir cuentas: `VaultState`, `UserBonus` con discriminadores y tamaños acotados.
3. Implementar derivación y constantes de seeds alineadas con `design.md`.
4. Implementar `initialize_vault` (crear state + vault token account + authority PDA según diseño).
5. Implementar `register_user` con verificación de admin y persistencia de `amount_allocated` (definir política init/update y documentarla en código).
6. Implementar `claim` con CPI SPL Token y actualización de `amount_claimed`.
7. Añadir eventos o logs mínimos útiles para depuración en localnet.
8. Ejecutar `anchor build` y corregir clippy/lints relevantes.

## Tests (Anchor + local)

9. Test: admin inicializa vault; mint y vault token account coherentes.
10. Test: admin registra usuario con monto N; PDA usuario derivable off-chain igual que on-chain.
11. Test: usuario claim parcial y total; fallos esperados (exceso, wrong mint, wrong owner).
12. Documentar en comentario o README cómo lanzar `solana-test-validator` y cargar el programa.

## Off-chain (cliente / frontend)

13. Crear app (por ejemplo Vite + React + TypeScript) o carpeta `app/` con dependencias Solana + wallet-adapter.
14. Configurar `Connection` a `http://localhost:8899`.
15. Integrar **Backpack** en el selector de wallets del adapter.
16. Implementar llamadas Anchor IDL: `initializeVault`, `registerUser`, `claim` con cuentas y PDAs correctas.
17. Flujo admin: crear/obtener mint local, financiar vault, registrar usuarios de prueba.
18. Flujo usuario: conectar wallet, ejecutar claim contra programa desplegado en localnet.

## Deploy / operaciones (localnet)

19. Script o instrucciones: `solana program deploy` contra localnet con la clave del programa generada por Anchor.
20. Verificar `declare_id!` / `Program ID` en `Anchor.toml` y en el cliente coinciden tras deploy.

## Entrega

21. Revisar que README o sección de docs describa: arranque validator, deploy, fund vault, flujo admin/usuario y puerto 8899.
