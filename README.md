
```markdown
# Sistema de Geração de Referências — Pagamento Antecipado de Propinas

Este repositório contém o código-fonte do formulário web personalizado para a automatização e geração de dados de pagamento (Entidade, Referência e Valor) destinados ao pagamento antecipado de propinas. A solução foi desenhada para ser hospedada estaticamente no **GitHub Pages** e incorporada de forma nativa na intranet corporativa em **Microsoft SharePoint Online** através de um `iframe`.

---

## 1. Visão Geral da Solução

A arquitectura da solução é minimalista, segura e distribuída em três componentes principais que comunicam entre si sem a necessidade de servidores dedicados de backend:

1. **Frontend (GitHub Pages):** Interface de utilizador desenvolvida em HTML5, CSS3 e JavaScript puro (Vanilla JS). É responsável por recolher as selecções, calcular os descontos progressivos e apresentar os dados finais de pagamento.
2. **Middleware / Backend (Power Automate):** Actua como o motor de integração (API). Recebe as requisições do formulário, valida os tokens de segurança, comunica com as listas de dados no SharePoint, solicita a geração de referências ao gateway de pagamento da rede Multicaixa e despoleta a notificação por e-mail.
3. **Plataforma de Apresentação (SharePoint Online):** Centraliza o acesso dos utilizadores. O formulário é embutido numa página interna através de um `iframe`, isolando o ambiente de execução mas garantindo uma integração visual perfeita (Fluent Design).

---

## 2. Funcionalidades Implementadas

* **Mecanismo de Confirmação Inicial:** Apresentação de um pop-up obrigatório com efeito de desfoque de fundo (*backdrop blur*) antes da inicialização do formulário. Evita chamadas acidentais ou automáticas à API, protegendo os limites de execução do Power Automate.
* **Carregamento Dinâmico (initFetch):** Comunicação imediata com o Power Automate após a confirmação do utilizador para obter os dados actualizados de turmas, classes e propinas.
* **Dropdowns em Cascata:** Filtros inteligentes encadeados que guiam o utilizador na selecção correcta: **Classe** $\rightarrow$ **Turma** $\rightarrow$ **Nome do Aluno**.
* **Pesquisa Integrada:** Caixa de pesquisa em tempo real inserida dentro dos dropdowns customizados para facilitar a localização de alunos em listagens extensas.
* **Selecção Múltipla de Meses:** Permite seleccionar um ou vários meses do ano lectivo, incluindo um atalho para seleccionar automaticamente todo o ano corrente.
* **Cálculo de Desconto Progressivo:** Motor de cálculo interno que avalia o número de meses seleccionados e aplica a percentagem de desconto configurada para o perfil do aluno.
* **Campos Dinâmicos de Apenas Leitura:** Atualização instantânea dos campos de controlo: *Desconto Aplicado (%)*, *Valor do Desconto* e *Valor Total a Pagar*.
* **Formatação Monetária Angolana:** Todos os valores monetários são exibidos no padrão de leitura de Angola (ex: `12.500,00 Kz`).
* **Painel de Sucesso Georreferenciado:** Exibição imediata da Entidade, Referência, Valor e a nota de validade logo após a submissão. O painel aterra fixo a `250px` do topo, garantindo visibilidade total na área útil do *iframe* sem necessidade de scroll.
* **Consistência Cromática (image_fa8f86.png):** O valor monetário final no painel de sucesso adota rigorosamente a cor cinza-escuro padrão (`#323130`), unificando a leitura visual com os campos de Entidade e Referência.
* **Gestão de Estados (Loading & Erros):** Bloqueio completo dos controlo do formulário durante as chamadas assíncronas e exibição de mensagens de erro amigáveis em caso de falhas de comunicação.

---

## 3. Estrutura do Repositório

O repositório mantém uma estrutura estática simples e de fácil manutenção:

```text
├── index.html          # Ficheiro principal contendo a estrutura HTML, estilos CSS e lógica JS.
└── README.md           # Este documento de especificação e guias do projecto.

```

* **`index.html`**: Centraliza todo o código para garantir que o carregamento no SharePoint seja o mais rápido possível, eliminando requisições adicionais a ficheiros `.css` ou `.js` externos.

---

## 4. Payloads de Comunicação (Power Automate)

### 4.1. Inicialização do Formulário (`initFetch`)

Disparado quando o utilizador clica em "Sim, continuar" no pop-up de entrada.

**Pedido (POST):**

```json
{
  "Flag": "GetSPOData-AnteciparPropina",
  "Source": "[CHAVE_DE_AUTENTICAÇÃO_RESTRITA]"
}

```

---

### 4.2. Submissão do Formulário e Requisição de Referência

Disparado ao clicar no botão verde "Gerar Dados de Pagamento". O payload extrai o identificador numérico real do aluno (`NumeroAluno`) devolvido no arranque e contabiliza os metadados financeiros calculados em memória.

**Pedido (POST):**

```json
{
  "Flag": "GetPaymentReference",
  "Servico": "Propina-Antecipada",
  "Classe": "3ª Classe",
  "Turma": "3-B",
  "NomeDoAluno": "Brígida Pinto",
  "NumeroAluno": "8",
  "ID": "38",
  "Telefone": "930803636",
  "Email": "encarregado@exemplo.com",
  "ValorTotal": "387000.00",
  "ValorDesconto": "12500.00",
  "EntidadeMcx": "30067",
  "Notes": "Observações textuais opcionais introduzidas pelo utilizador",
  "MesAntecipado": "Janeiro,Fevereiro,Março",
  "NumMesesAnticipados": "3",
  "Source": "[CHAVE_DE_AUTENTICAÇÃO_RESTRITA]"
}

```

**Resposta Esperada (200 OK):**

```json
{
  "Entidade": "30067",
  "Referencia": "922273105",
  "Valor": 387000,
  "Validade": "18-07-2026"
}

```

---

## 5. Incorporação no SharePoint Online

Para embutir este formulário numa página do SharePoint, siga os passos abaixo:

1. Coloque a página do SharePoint em modo de **Edição**.
2. Adicione a Web Part **Embutir / Embed** (`</>`).
3. No painel de propriedades à direita, cole o código do `iframe` adaptado:

```html
<iframe src="https://[organizacao].github.io/[nome-do-repositorio]/" 
        width="100%" 
        height="780px" 
        style="border: none; background: transparent; overflow-y: auto;" 
        scrolling="yes">
</iframe>

```

> **Nota de Configuração:** A propriedade `height="780px"` associada à ancoragem absoluta a `250px` do painel de sucesso garante que todo o desfecho da operação seja assistido pelo utilizador logo no primeiro carregamento visual do SharePoint.

---

## 6. Como Actualizar o Formulário

Sempre que for necessário efetuar alterações no design ou na lógica do formulário, siga o fluxo padrão:

1. Aceda ao ficheiro `index.html` neste repositório.
2. Clique no **ícone do lápis** (Editar) no canto superior direito.
3. Aplique as alterações necessárias no código.
4. Clique em **Commit changes...**, introduza uma mensagem descritiva da alteração e confirme o *commit* na ramificação principal (`main` ou `master`).
5. O GitHub Pages irá reconstruir a página automaticamente em cerca de 1 a 2 minutos. As alterações serão refletidas de imediato no SharePoint de forma totalmente transparente para os utilizadores.

---

## 7. Considerações Técnicas

* **CORS (Cross-Origin Resource Sharing):** O Power Automate foi configurado no seu gatilho HTTP para aceitar requisições vindas do domínio público do GitHub Pages (`github.io`). Caso o URL do repositório seja alterado, o administrador do fluxo deverá atualizar as políticas de cabeçalho no Power Automate.
* **Segurança:** O campo `Source` enviado nos payloads funciona como uma chave de segurança por assinatura de origem. O repositório deve ser mantido como **Público** para que o GitHub Pages funcione gratuitamente, pelo que nenhuma credencial confidencial de bases de dados ou chaves mestras de API devem ser escritas diretamente no código do formulário.
* **Independência Tecnológica:** A página foi desenvolvida sem recurso a frameworks pesadas (como React, Angular ou Vue) ou bibliotecas de terceiros (como jQuery). Isto garante um tempo de carregamento inferior a 500ms e elimina o risco de quebras por atualizações de dependências externas de pacotes.

---

## 8. Escopo do Serviço

> [!IMPORTANT]
> Este formulário e este repositório são **exclusivos e dedicados** ao serviço de **Propina-Antecipada**.
> Outros serviços da instituição (como matrículas, taxas de exames ou emissão de declarações) seguem regras de negócio, tabelas de descontos e fluxos de aprovação distintos. Por esse motivo, serão geridos em formulários independentes e alojados em repositórios separados para garantir a modularidade e integridade do sistema.

```

```
