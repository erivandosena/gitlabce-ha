# Testes Funcionais do GitLab

Este documento descreve os testes para validar as funcionalidades do GitLab implantado no Kubernetes.

---

## Testes de Funcionalidades Básicas

### Teste 1: Acesso à Interface Web
1. Acesse o endereço do GitLab configurado no Ingress (exemplo: `http://gitlab.local`).
2. Verifique se a página de login é exibida corretamente.
3. Realize login com as credenciais do administrador criadas no processo de instalação.

### Teste 2: Criação de Projetos
1. No painel do GitLab, clique em **New Project**.
2. Preencha os campos obrigatórios (nome do projeto, visibilidade).
3. Clique em **Create Project**.
4. Verifique se o projeto foi criado com sucesso.

### Teste 3: Push para Repositório
1. Clone o repositório criado localmente:
   ```bash
   git clone http://gitlab.local/<group>/<project>.git
