# 🚀 Guia Completo: Conectando seu Repositório ao Azure DevOps

## 📋 Pré-requisitos

Antes de começar, certifique-se de ter:
- ✅ Conta Microsoft (Outlook, Hotmail, ou corporativa)
- ✅ Seu código em um repositório Git (GitHub, GitLab, Bitbucket, ou local)
- ✅ Acesso de administrador ao projeto (se corporativo)

---

## 🎯 Cenário 1: Repositório no GitHub (Mais Comum)

### Passo 1: Criar Conta no Azure DevOps
1. Acesse: **https://dev.azure.com**
2. Clique em **"Start free"**
3. Entre com sua conta Microsoft
4. Escolha um nome para sua organização (ex: `seu-nome-devops`)

### Passo 2: Criar um Novo Projeto
1. No Azure DevOps, clique em **"New Project"**
2. Preencha os dados:
   ```
   Project name: PGATS-CI-Lab
   Description: Projeto de Integração Contínua para PGATS
   Visibility: Private (recomendado para estudos)
   ```
3. Clique em **"Create"**

### Passo 3: Conectar com GitHub
1. No seu projeto, vá para **"Pipelines"** → **"New Pipeline"**
2. Selecione **"GitHub"**
3. **Autorize o Azure DevOps** a acessar sua conta GitHub:
   - Clique em **"Authorize AzurePipelines"**
   - Digite sua senha do GitHub se solicitada
   - Confirme as permissões

### Passo 4: Selecionar o Repositório
1. Escolha seu repositório **`pgats-ci-lab`** na lista
2. Se não aparecer, clique em **"All repositories"** para ver todos

### Passo 5: Configurar o Pipeline
1. Selecione **"Existing Azure Pipelines YAML file"**
2. Escolha a branch: **`main`** ou **`master`**
3. Selecione o caminho: **`/azure-pipelines.yml`**
4. Clique em **"Continue"**

### Passo 6: Revisar e Executar
1. Revise o conteúdo do pipeline
2. Clique em **"Run"** para executar pela primeira vez
3. Aguarde a conclusão (pode levar 10-15 minutos)

---

## 🎯 Cenário 2: Repositório Local ou Outro Provedor

### Opção A: Migrar para Azure Repos

#### Passo 1: Criar Repositório no Azure Repos
1. No seu projeto Azure DevOps, vá para **"Repos"**
2. Clique em **"Import a repository"**
3. Cole a URL do seu repositório atual
4. Clique em **"Import"**

#### Passo 2: Clonar Localmente (Opcional)
```bash
# Clone o novo repositório Azure
git clone https://dev.azure.com/sua-org/PGATS-CI-Lab/_git/PGATS-CI-Lab

# Copie os arquivos do seu projeto atual
# Commit e push as mudanças
git add .
git commit -m "Migração inicial para Azure DevOps"
git push origin main
```

#### Passo 3: Criar Pipeline
1. Vá para **"Pipelines"** → **"New Pipeline"**
2. Selecione **"Azure Repos Git"**
3. Escolha seu repositório
4. Selecione **"Existing Azure Pipelines YAML file"**
5. Escolha `/azure-pipelines.yml`
6. Clique em **"Run"**

### Opção B: Manter no GitHub/GitLab
Se preferir manter no GitHub:
1. Siga os **Passos 1-6 do Cenário 1**
2. Autorize as permissões necessárias

---

## 🔧 Configurações Avançadas

### Configurar Service Connections (Se Necessário)

#### Para GitHub:
1. Vá para **"Project Settings"** → **"Service connections"**
2. Clique em **"New service connection"**
3. Selecione **"GitHub"**
4. Escolha **"Grant authorization"**
5. Autorize e configure

#### Para outros provedores:
```
• GitLab: Use "Generic Git" ou "GitLab"
• Bitbucket: Use "Bitbucket Cloud"
• Azure Repos: Configuração automática
```

### Configurar Branch Policies (Recomendado)
1. Vá para **"Repos"** → **"Branches"**
2. Clique nos **"..."** da branch `main`
3. Selecione **"Branch policies"**
4. Configure:
   ```
   ✅ Require a minimum number of reviewers: 1
   ✅ Check for linked work items: On
   ✅ Check for comment resolution: On
   ✅ Build validation: Adicione seu pipeline
   ```

---

## 📊 Verificando a Configuração

### 1. Teste o Pipeline
```bash
# Faça uma mudança no código
echo "# Teste Azure DevOps" >> README.md
git add README.md
git commit -m "feat: teste pipeline Azure DevOps"
git push origin main
```

### 2. Monitore a Execução
1. Vá para **"Pipelines"**
2. Clique no pipeline em execução
3. Acompanhe cada estágio:
   - ✅ Setup e Preparação
   - ✅ Verificação de Qualidade  
   - ✅ Testes Unitários
   - ✅ Testes de Mutação
   - ✅ Testes End-to-End
   - ✅ Build e Deploy

### 3. Verifique os Relatórios
1. **Test Results**: Resultados dos testes
2. **Code Coverage**: Cobertura de código
3. **Artifacts**: Arquivos de build
4. **Extensions**: Relatórios adicionais

---

## 🚨 Troubleshooting Comum

### ❌ Erro: "Repository not found"
**Solução:**
```bash
1. Verifique se o repositório existe
2. Confirme as permissões de acesso
3. Re-autorize a conexão com GitHub
```

### ❌ Erro: "YAML file not found"
**Solução:**
```bash
1. Confirme que o arquivo azure-pipelines.yml está na raiz
2. Verifique se está na branch correta
3. Commit e push o arquivo se necessário
```

### ❌ Erro: "Permission denied"
**Solução:**
```bash
1. Vá para Project Settings → Permissions
2. Adicione-se como Administrator
3. Configure as permissões de pipeline
```

### ❌ Pipeline falha nos testes
**Solução:**
```bash
1. Execute os testes localmente primeiro:
   yarn install
   yarn run test
   yarn run e2e

2. Verifique os logs no Azure DevOps
3. Ajuste timeouts se necessário
```

---

## 🎓 Configurações para Aprendizado

### Para Ambiente de Estudos:
```yaml
# Desabilitar alguns estágios para economizar tempo
- stage: MutationTests
  condition: eq(variables['Build.Reason'], 'Manual')  # Só manual
```

### Para Demonstrações:
```yaml
# Adicionar mais logs verbose
- script: |
    echo "=== EXECUTANDO TESTES ==="
    yarn run test --verbose
    echo "=== TESTES CONCLUÍDOS ==="
```

---

## 📈 Métricas e Monitoramento

### Dashboard Recomendado:
1. **Pipelines**: Taxa de sucesso
2. **Test Results**: Cobertura de código
3. **Artifacts**: Tamanho dos builds
4. **Deployment**: Frequência de releases

### Alertas Importantes:
- 🚨 Pipeline falhando > 2x seguidas
- 🚨 Cobertura < 80%
- 🚨 Tempo execução > 20min
- 🚨 Testes E2E instáveis

---

## 🔗 Links Úteis

### Documentação:
- **Azure DevOps**: https://docs.microsoft.com/azure/devops/
- **YAML Schema**: https://docs.microsoft.com/azure/devops/pipelines/yaml-schema
- **GitHub Integration**: https://docs.microsoft.com/azure/devops/pipelines/repos/github

### Extensões Recomendadas:
- **SonarQube**: Análise de código
- **WhiteSource**: Segurança de dependências
- **Marketplace**: https://marketplace.visualstudio.com/azuredevops

---

## ✅ Checklist de Configuração

### Antes de Executar:
- [ ] Conta Azure DevOps criada
- [ ] Projeto configurado
- [ ] Repositório conectado
- [ ] Pipeline YAML commitado
- [ ] Permissões configuradas

### Após Primeira Execução:
- [ ] Pipeline executado com sucesso
- [ ] Relatórios publicados
- [ ] Artefatos gerados
- [ ] Branch policies configuradas
- [ ] Notificações ativadas

### Para Produção:
- [ ] Service connections seguras
- [ ] Secrets configurados
- [ ] Environment approvals
- [ ] Deployment gates
- [ ] Monitoring ativo

---

**🎯 Resultado Final:**
Um pipeline completo de CI/CD executando automaticamente a cada commit, garantindo qualidade e entrega contínua do seu projeto PGATS! 

💜⚡️ **Criado para PGATS - Pós-Graduação em Automação de Testes**