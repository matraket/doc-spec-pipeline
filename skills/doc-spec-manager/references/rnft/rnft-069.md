> **Categoría:** 6. Mantenibilidad y Operaciones

### 6.7 RNFT-069: Consistencia Cross-Tab del Tenant Activo (frontend)

**RNF Base:** RNF-069 (Consistencia Cross-Tab del Tenant Activo)

**Contexto:** Cuando un switch-tenant ocurre en una pestaña, las demás pestañas del mismo origin con la aplicación abierta quedan con `tenantId` stale. El JWT compartido vía `localStorage`/`sessionStorage` pasa a ser del nuevo tenant pero la UI sigue renderizada con el contexto viejo. La próxima request autenticada de la pestaña stale cargaría datos del nuevo tenant sobre la UI del viejo (riesgo de fuga cross-tenant visible al usuario).

**Implementación de referencia (React + auth provider):**

El frontend detecta cambios del tenant activo entre pestañas vía `BroadcastChannel('auth')` y renderiza un banner persistente de resolución que bloquea la operación hasta que el usuario tome una acción explícita.

```typescript
// auth-provider.tsx (extracto conceptual)
const channel = new BroadcastChannel('auth');

// Al completar un switch-tenant o logout:
function onTenantChanged(newTenantId: string | null, newTenantName?: string) {
  channel.postMessage({
    type: 'tenant-changed',
    tenantId: newTenantId,
    tenantName: newTenantName,
    at: Date.now(),
  });
}

// Al recibir notificación de otra pestaña:
channel.onmessage = (event) => {
  const { type, tenantId: incomingTenantId, tenantName } = event.data;
  if (type !== 'tenant-changed') return;

  const currentTenantId = getActiveTenantIdFromState();
  if (incomingTenantId !== currentTenantId) {
    // Banner persistente, sin dismiss
    showTenantDriftBanner({
      message: `El tenant activo cambió a ${tenantName} en otra pestaña.`,
      actions: [
        { label: `Recargar en ${tenantName}`, onClick: () => window.location.reload() },
        { label: 'Cerrar sesión',              onClick: () => logout() },
      ],
      dismissable: false,
    });
    // Bloquear requests nuevas mientras el banner está activo
    apiClient.pauseOutgoingRequests();
  }
};
```

**Criterios de verificación:**

- Test E2E con dos pestañas abiertas: switch-tenant en pestaña 1 → pestaña 2 muestra banner en <1s, banner no se puede cerrar sin acción, la siguiente request de pestaña 2 no parte hasta que el usuario resuelve.
- Test unit del auth provider: recibir mensaje `tenant-changed` con `tenantId` distinto al actual dispara render del banner; con el mismo tenantId no hace nada.

**Alternativas aceptables:**

- `storage` events (`window.addEventListener('storage', ...)`) en entornos donde `BroadcastChannel` no esté disponible.
- Polling server-side (`GET /auth/me`) como fallback para navegadores sin BroadcastChannel ni storage events.

**Trazabilidad:** RNF-069, ADR-006 (amendment Abr 2026), UC-002 FA-2.
