# Capability: Bonus vault (SPL)

## initialize_vault

**Given** un mint SPL existente y una clave admin  
**When** el admin invoca `initialize_vault` con las cuentas correctas  
**Then** se crea el estado del vault, la token account del pool queda bajo authority PDA y el mint queda registrado en el estado

## register_user

**Given** un vault inicializado y el firmante es el admin del vault  
**When** el admin invoca `register_user` con un usuario U y monto M  
**Then** existe el PDA de bonus para U con `amount_allocated == M` (según política init/update definida en implementación)

## register_user — no admin

**Given** un vault inicializado y el firmante no es el admin  
**When** intenta `register_user`  
**Then** la transacción falla

## claim

**Given** un usuario U con asignación A y reclamado C, y fondos suficientes en la vault token account  
**When** U invoca `claim` por un monto válido ≤ (A − C)  
**Then** los tokens se transfieren a la ATA de U y aumenta el reclamado acumulado

## claim — exceso

**Given** pendiente P = A − C  
**When** U intenta reclamar más de P  
**Then** la transacción falla

## claim — mint incorrecto

**Given** cuentas de token con mint distinto al del vault  
**When** se invoca `claim`  
**Then** la transacción falla
