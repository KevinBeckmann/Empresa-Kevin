# Firebase Communication Implementation

## Visão Geral

Este documento descreve a implementação da comunicação em tempo real entre `index.html` e `admin.html` usando Firebase Firestore.

## Arquitetura

### Coleções Firestore

1. **`siteProjects`** - Armazena todos os projetos do site
   - Campos: `title`, `description`, `category`, `status`, `order`, `features`, `images`, `timestamp`, `updatedAt`
   - Status possíveis: `active`, `draft`, `archived`
   - Apenas projetos com `status: "active"` são exibidos no site público

2. **`adminActions`** - Log de ações administrativas para comunicação entre páginas
   - Campos: `action`, `projectId`, `projectTitle`, `timestamp`, `user`
   - Ações: `created_project`, `updated_project`, `activated_project`, `archived_project`

### Fluxo de Comunicação

```
┌─────────────────┐                    ┌──────────────────┐
│   admin.html    │                    │   index.html     │
│                 │                    │                  │
│  1. Admin cria/ │                    │                  │
│     edita       │                    │                  │
│     projeto     │                    │                  │
│        ↓        │                    │                  │
│  2. Salva no    │──────────────────→ │                  │
│     Firestore   │   Firebase         │                  │
│     (siteProje- │   Realtime         │  3. onSnapshot   │
│     cts)        │   Sync             │     detecta      │
│        ↓        │                    │     mudança      │
│  4. Registra    │                    │        ↓         │
│     ação em     │──────────────────→ │  5. Atualiza     │
│     adminActio  │   Firebase         │     lista auto   │
│     ns          │   Realtime         │        ↓         │
│                 │   Sync             │  6. Mostra       │
│                 │                    │     notificação  │
└─────────────────┘                    └──────────────────┘
```

## Funcionalidades Implementadas

### 1. Sincronização em Tempo Real (Real-time Sync)

**admin.html:**
- Usa `onSnapshot()` no lugar de `getDocs()` para monitorar mudanças na coleção `siteProjects`
- Quando qualquer admin faz alterações, todos os painéis admin conectados são atualizados automaticamente
- Indicador de status: "sincronizado" (verde), "erro" (vermelho), ou "conectando..." (amarelo)

**index.html:**
- Usa `onSnapshot()` para monitorar projetos ativos
- Atualiza automaticamente a lista de projetos quando há mudanças
- Indicador visual "Sincronizado" no header

### 2. Log de Ações (Action Logging)

Toda ação administrativa é registrada na coleção `adminActions`:
- **created_project** - Novo projeto criado
- **updated_project** - Projeto atualizado
- **activated_project** - Projeto reativado
- **archived_project** - Projeto arquivado

### 3. Notificações em Tempo Real

**index.html** monitora a coleção `adminActions` e exibe notificações quando:
- Um novo projeto é criado
- Um projeto é atualizado
- Um projeto é ativado ou arquivado

As notificações aparecem no canto superior direito com animação e desaparecem após 3 segundos.

### 4. Indicadores Visuais

**admin.html:**
```
┌─────────────────────────────────────────────────┐
│  Firestore: sincronizado  [Recarregar] [⚙] [🔧] │
└─────────────────────────────────────────────────┘
```

**index.html:**
```
┌──────────────────────────────────────┐
│  [⚙] [● Sincronizado]                │
└──────────────────────────────────────┘
```

## Código Principal

### index.html - Listener de Projetos

```javascript
// Carrega projetos com real-time sync
function loadProjectsFromFirestore() {
    const projectsCol = collection(db, "siteProjects");
    const q = query(
        projectsCol,
        where("status", "==", "active"),
        orderBy("timestamp", "desc")
    );
    
    // onSnapshot dispara automaticamente quando há mudanças
    projectsUnsubscribe = onSnapshot(q, (snapshot) => {
        const projects = [];
        snapshot.forEach((doc) => {
            projects.push({ id: doc.id, ...doc.data() });
        });
        renderProjects(projects);
        updateSyncIndicator(true);
    });
}
```

### index.html - Monitor de Ações

```javascript
// Monitora ações do admin
function monitorAdminActions() {
    const actionsCol = collection(db, "adminActions");
    const q = query(actionsCol, orderBy("timestamp", "desc"));
    
    adminActionsUnsubscribe = onSnapshot(q, (snapshot) => {
        snapshot.docChanges().forEach((change) => {
            if (change.type === "added") {
                const action = change.doc.data();
                showSyncNotification(action);
            }
        });
    });
}
```

### admin.html - Salvar Projeto

```javascript
// Ao salvar, registra a ação
await updateDoc(ref, baseData);
await logAdminAction("updated_project", existingId, title);
```

### admin.html - Log de Ações

```javascript
async function logAdminAction(action, projectId, projectTitle) {
    const actionsCol = collection(db, "adminActions");
    await addDoc(actionsCol, {
        action: action,
        projectId: projectId,
        projectTitle: projectTitle,
        timestamp: serverTimestamp(),
        user: "admin"
    });
}
```

## Segurança

### XSS Protection

Todos os dados vindos do Firestore passam pela função `escapeHtml()` antes de serem renderizados:

```javascript
function escapeHtml(text) {
    if (text == null) return '';
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
}
```

### Limpeza de Listeners

Listeners são automaticamente removidos quando a página é fechada:

```javascript
window.addEventListener('beforeunload', () => {
    if (projectsUnsubscribe) {
        projectsUnsubscribe();
    }
    if (adminActionsUnsubscribe) {
        adminActionsUnsubscribe();
    }
});
```

## Configuração do Firebase

O projeto usa a seguinte configuração Firebase (já configurada no código):

```javascript
const firebaseConfig = {
    apiKey: "AIzaSyAqNKa5jp5UpcTtnAF1_EUVKCXVrGZTFkA",
    authDomain: "bemorh-bc5a8.firebaseapp.com",
    projectId: "bemorh-bc5a8",
    storageBucket: "bemorh-bc5a8.firebasestorage.app",
    messagingSenderId: "313271255362",
    appId: "1:313271255362:web:167e42b830f8fdcb9188db"
};
```

## Testes

### Testar Comunicação em Tempo Real

1. Abra `index.html` em uma aba do navegador
2. Abra `admin.html` em outra aba
3. No admin, crie ou edite um projeto com status "Ativo"
4. Observe no `index.html`:
   - A lista de projetos é atualizada automaticamente
   - Uma notificação aparece no canto superior direito
   - O indicador "Sincronizado" permanece ativo

### Verificar Logs

Abra o Console do navegador (F12) para ver:
- `✓ Projetos sincronizados em tempo real: X`
- `✓ Ação registrada: created_project`

## Benefícios

1. **Atualizações Instantâneas**: Mudanças aparecem imediatamente sem refresh
2. **Comunicação Bidirecional**: Admin e site público se comunicam através do Firebase
3. **Escalabilidade**: Suporta múltiplos admins editando simultaneamente
4. **Experiência do Usuário**: Notificações informam sobre mudanças
5. **Confiabilidade**: Firebase gerencia conexões e reconexões automaticamente

## Limitações e Melhorias Futuras

### Limitações Atuais
- Autenticação local (localStorage) - não é segura para produção
- Sem controle de permissões granular
- Limite de leituras/escritas do Firestore (plano gratuito)

### Melhorias Sugeridas
1. Implementar Firebase Authentication para autenticação real
2. Adicionar Firebase Security Rules para controle de acesso
3. Implementar paginação para grandes volumes de dados
4. Adicionar indicador de "outro admin está editando este projeto"
5. Sistema de notificações mais robusto (push notifications)
6. Cache offline com Firebase Offline Persistence

## Referências

- [Firebase Firestore Documentation](https://firebase.google.com/docs/firestore)
- [Real-time Updates with onSnapshot](https://firebase.google.com/docs/firestore/query-data/listen)
- [Firebase Security Rules](https://firebase.google.com/docs/firestore/security/get-started)
