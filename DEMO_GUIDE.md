# Guia de Demonstração - Comunicação Firebase

## Como Testar a Comunicação em Tempo Real

### Pré-requisitos
- Navegador moderno (Chrome, Firefox, Edge)
- Conexão com internet (para acessar Firebase CDN)

### Passo a Passo

#### 1. Abrir as Duas Páginas

**Opção A: Localmente**
```bash
# Inicie um servidor HTTP local
python3 -m http.server 8080

# Ou use Node.js
npx serve
```

Depois abra:
- Aba 1: `http://localhost:8080/index.html`
- Aba 2: `http://localhost:8080/admin.html`

**Opção B: GitHub Pages ou Hosting**
- Aba 1: URL do site público
- Aba 2: URL do painel admin

#### 2. Fazer Login no Admin

1. Na aba do `admin.html`, você já está no painel
2. Observe o indicador no topo: "Firestore: sincronizado" (em azul/verde)

#### 3. Criar um Novo Projeto

1. No painel admin, preencha o formulário "Novo Projeto":
   - **Título**: "Projeto de Teste Firebase"
   - **Categoria**: "Desenvolvimento Web"
   - **Status**: "Ativo (aparece no site)"
   - **Descrição**: "Este projeto testa a comunicação em tempo real"
   - **Recursos**: "Real-time sync, Firebase Firestore"

2. Clique em **Salvar**

3. **RESULTADO ESPERADO**:
   - ✅ Projeto aparece imediatamente na tabela do admin
   - ✅ Na aba do `index.html`, o projeto aparece automaticamente
   - ✅ Uma notificação verde aparece no canto superior direito do index:
     ```
     ✓ Novo projeto adicionado: Projeto de Teste Firebase
     ```

#### 4. Editar um Projeto

1. No admin, clique no ícone de editar (✏️) de um projeto
2. Altere o título para "Projeto ATUALIZADO"
3. Clique em **Salvar**

4. **RESULTADO ESPERADO**:
   - ✅ Mudança aparece instantaneamente na tabela do admin
   - ✅ No `index.html`, o card do projeto é atualizado automaticamente
   - ✅ Notificação aparece: "✓ Projeto atualizado: Projeto ATUALIZADO"

#### 5. Arquivar um Projeto

1. No admin, clique no ícone de arquivar (📦) de um projeto ativo
2. Confirme a ação

3. **RESULTADO ESPERADO**:
   - ✅ Status muda para "Arquivado" no admin
   - ✅ No `index.html`, o projeto desaparece automaticamente da lista
   - ✅ Notificação: "✓ Projeto arquivado"

#### 6. Reativar um Projeto

1. No admin, clique no ícone de restaurar (↺) de um projeto arquivado
2. Confirme a ação

3. **RESULTADO ESPERADO**:
   - ✅ Status muda para "Ativo" no admin
   - ✅ No `index.html`, o projeto reaparece automaticamente
   - ✅ Notificação: "✓ Projeto reativado"

### Verificações Visuais

#### Indicadores de Sincronização

**admin.html:**
```
┌─────────────────────────────────────────────┐
│ Gestão de Projetos                          │
│ Firestore: sincronizado [Recarregar] [⚙][🔧]│
└─────────────────────────────────────────────┘
```
- Verde/Azul = Conectado e sincronizado
- Amarelo = Conectando...
- Vermelho = Erro de conexão

**index.html:**
```
┌──────────────────────────────────────┐
│ [Logo] Bemorh  [Nav...]  [⚙] [●Sincronizado] │
└──────────────────────────────────────┘
```
- Verde = Sincronizado com Firebase
- Vermelho = Desconectado

#### Notificações (index.html)

Quando o admin faz uma ação, aparece no canto superior direito:

```
┌─────────────────────────────────────┐
│ 🔄 ✓ Novo projeto adicionado: [...]  │
└─────────────────────────────────────┘
```

A notificação:
- Desliza da direita para a esquerda
- Permanece por 3 segundos
- Some com animação

### Console do Navegador

Abra o DevTools (F12) > Console para ver logs:

**index.html:**
```
✓ Projetos sincronizados em tempo real: 3
✓ Dados sincronizados
```

**admin.html:**
```
✓ Projetos sincronizados em tempo real: 5
✓ Ação registrada: created_project
✓ Ação registrada: updated_project
```

### Teste de Múltiplos Admins

1. Abra `admin.html` em **duas abas diferentes**
2. Na aba 1, crie um projeto
3. **RESULTADO**: A aba 2 mostra o novo projeto imediatamente
4. Na aba 2, edite esse projeto
5. **RESULTADO**: A aba 1 vê a edição instantaneamente

### Teste de Persistência

1. Crie/edite projetos no admin
2. Feche e reabra o `index.html`
3. **RESULTADO**: Todos os projetos carregam do Firebase automaticamente

### Troubleshooting

#### "Firestore: conectando..." não muda

**Problema**: Não consegue conectar ao Firebase

**Soluções**:
- Verifique sua conexão com internet
- Verifique se o Firebase CDN não está bloqueado
- Abra o Console e procure por erros
- Verifique as credenciais do Firebase

#### Projetos não aparecem no index.html

**Problema**: Lista vazia no site público

**Possíveis causas**:
- Os projetos no admin estão com status "Rascunho" ou "Arquivado"
- Problema de permissões no Firestore
- O listener não está funcionando

**Solução**:
1. Certifique-se que os projetos têm status "Ativo"
2. Recarregue a página
3. Verifique o Console por erros

#### Notificações não aparecem

**Problema**: Ações no admin não geram notificações no index

**Possíveis causas**:
- O timestamp da ação é muito antigo
- O listener de adminActions não está ativo

**Solução**:
1. Recarregue o index.html
2. Verifique o Console: `monitorAdminActions` deve estar ativo
3. Crie um novo projeto (ações muito antigas não geram notificação)

### Dados de Teste

Use estes dados para criar projetos de teste:

**Projeto 1:**
- Título: Sistema de Gestão Empresarial
- Categoria: Desenvolvimento Web
- Status: Ativo
- Descrição: Plataforma completa para gestão empresarial com dashboards interativos
- Recursos: Dashboard em tempo real, Relatórios automáticos, API RESTful

**Projeto 2:**
- Título: App Mobile de Delivery
- Categoria: Apps Mobile
- Status: Ativo
- Descrição: Aplicativo nativo para iOS e Android com rastreamento em tempo real
- Recursos: GPS tracking, Pagamento integrado, Push notifications

**Projeto 3:**
- Título: Design System Corporativo
- Categoria: UI/UX Design
- Status: Ativo
- Descrição: Sistema de design completo com componentes reutilizáveis
- Recursos: Biblioteca de componentes, Guia de estilos, Documentação interativa

## Conclusão

A comunicação Firebase está funcionando quando:
- ✅ Indicadores mostram "sincronizado"
- ✅ Mudanças no admin aparecem instantaneamente no index
- ✅ Notificações aparecem no index quando admin faz ações
- ✅ Múltiplas abas do admin sincronizam entre si
- ✅ Dados persistem após recarregar a página
