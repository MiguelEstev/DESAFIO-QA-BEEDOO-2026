# DESAFIO QA BEEDOO 2026

**Aplicação testada:** https://creative-sherbet-a51eac.netlify.app/  

---

## 1. Análise inicial da aplicação

### Qual o objetivo da aplicação?
A aplicação é um sistema de cadastro e listagem de cursos, operando como um CRUD (mas sem update e delete funcionais). O objetivo é permitir que um usuário crie novos cursos (Online ou Presenciais) fornecendo os detalhes necessários, visualizando todos os cursos criados em uma vitrine de cards e podendo excluí-los quando necessário.

### Quais são os principais fluxos disponíveis?
Durante a exploração, mapeei os seguintes fluxos principais:
- **Fluxo de Cadastro (Create):** Acesso ao formulário `/new-course` e preenchimento de até 10 campos (variando entre campos de URL para cursos Online e Endereço para cursos Presenciais).
- **Fluxo de Listagem (Read):** Exibição dos cursos cadastrados na página inicial através de cards resumidos.
- **Fluxo de Exclusão (Delete):** Remoção de um curso da listagem através de um botão presente em cada card.

### Quais pontos do sistema considero mais críticos para teste?
1. **Validação de Fluxo de Inserção de Dados (Campos Críticos):** A ausência de limites ou validações do tipo e formato nos inputs numéricos e de data.
2. **Roteamento da página inicial e new course:** Como os dados estão sendo tratados localmente sem um backend robusto visível, é crítico testar atualizações de página, criação de sessão e links diretos.
3. **Fluxo de Exclusão:** Avaliar o comportamento do fluxo de exclusão (falta de confirmação e sincronia com backend).

---

## 2. Decisões tomadas para criação dos testes
Para montar a estratégia, misturei testes de exploração da plataforma com análise de valor limite e foquei bastante em quebrar os formulários (testes negativos).

- **Garantir o fluxo "correto":** O primeiro passo foi ver se o básico estava funcionando, pois os bugs mais alarmantes podem estar nessa parte básica.
- **Testes negativos pesados:** Como o formulário coleta muitos dados importantes (quantidade de vagas, datas), tentei forçar a quebra da aplicação enviando texto onde era número, valores negativos e datas invertidas. Queria ver se existia alguma limitação do sistema para dados "errados".
- **Olhando o DevTools:** Abri o DevTools durante os testes, vendo as abas Network, Console e Application > LocalStorage. Isso foi útil para entender o que estava sendo enviado nas requisições e pegar erros não visuais.
- **Testes de roteamento:** Por ser uma SPA hospedada no Netlify, criei cenários para testar links diretos e comportamento do navegador (atualizar a página, voltar), para ver se o estado da aplicação e os dados não se perdiam no caminho.

---

## 3. Explicação do meu raciocínio durante a análise
A minha linha de raciocínio principal foi: Tentar ver a parte da frente e de trás da aplicação e sempre pensar em como testar mais erros.

Por exemplo, quando cliquei em "EXCLUIR CURSO" pela primeira vez, subiu a notificação de sucesso na hora. Mas o card continuou lá. Após isso, tentei atualizar a página e, quando não mudou nada, fui direto na aba Network investigar e vi que a requisição tomou um erro 405 Method Not Allowed. Ou seja: o front-end apresentava uma mensagem que não correspondia com o back, exibindo a mensagem de sucesso antes mesmo de saber se o servidor realmente excluiu.

Usei a mesma lógica para o formulário. Quando vi que ele deixava enviar campos vazios, comecei a testar a lógica dos dados (tipo colocar a data de término de um curso em 2020 e a de início em 2025). Deu para perceber que faltam proteções básicas no front-end e o backend simulado acaba aceitando os dados sujos do jeito que chegam.

---

## 4. Cenários e casos de teste
No fim, fiz 21 cenários de teste, focando no fluxo principal, na listagem de cursos, nas validações negativas e em problemas de roteamento.

Segue abaixo a planilha com os meus testes:
🔗 **Planilha:** [https://docs.google.com/spreadsheets/d/1oPSZ-tYNs6IEnOO30qilfwxqSl7qp5H0_YT5b0sKP7g/edit?usp=sharing]

---

## 5. Relatório de bugs encontrados
Foram encontrados 15 bugs no total:

### Bugs Críticos (Impedem ações fundamentais)

**BUG-01 — Formulário aceita envio com campos vazios**
- **Severidade:** Crítica
- **Passos para Reproduzir:** 1. Acessar `/new-course` 2. Clicar em CADASTRAR sem preencher nada.
- **Resultado Atual:** Card em branco criado na listagem.
- **Resultado Esperado:** Validação nos campos obrigatórios e impedir cadastro.
- **Impacto:** Dados poluídos e cards quebrados.

**BUG-02 — Exclusão não funciona e não pede confirmação**
- **Severidade:** Crítica
- **Passos para Reproduzir:** 1. Clicar em EXCLUIR CURSO. 2. Observar que não há diálogo de confirmação. 3. Observar o toast verde. 4. Dar F5.
- **Resultado Atual:** Ação imediata sem confirmação. Notificação diz sucesso, mas a API retorna 405. Curso continua lá.
- **Resultado Esperado:** Diálogo de confirmação antes da ação e curso removido de fato.
- **Impacto:** Funcionalidade principal quebrada e risco de exclusão acidental.

### Bugs Altos

**BUG-03 — Vagas negativas aceitas**
- **Severidade:** Alta
- **Passos para Reproduzir:** 1. Preencher vagas com -5. 2. Submeter.
- **Resultado Atual:** Cadastra com -5 VAGAS.
- **Resultado Esperado:** Aceitar apenas inteiros positivos.
- **Impacto:** Dados impossíveis no sistema.

**BUG-04 — Data de fim pode ser anterior à de início**
- **Severidade:** Alta
- **Passos para Reproduzir:** 1. Definir início 2025-10-10 e fim 2025-09-10. 2. Submeter.
- **Resultado Atual:** Aceita sem alerta.
- **Resultado Esperado:** Bloquear ou alertar sobre inconsistência.
- **Impacto:** Curso com duração negativa.

**BUG-05 — Link Direto retorna 404**
- **Severidade:** Alta
- **Passos para Reproduzir:** 1. Acessar: `https://creative-sherbet-a51eac.netlify.app/new-course` direto.
- **Resultado Atual:** Netlify retorna Not Found.
- **Resultado Esperado:** Formulário deve carregar.
- **Impacto:** Links compartilhados e favoritos quebrados.

**BUG-06 — Campos só com espaços em branco aceitos**
- **Severidade:** Alta
- **Passos para Reproduzir:** 1. Preencher campos de texto só com espaços. 2. Submeter.
- **Resultado Atual:** Card fantasma criado.
- **Resultado Esperado:** Fazer trim e tratar como vazio.
- **Impacto:** Ignora regra de preenchimento.

**BUG-07 — Link de inscrição coletado mas não exibido no card**
- **Severidade:** Alta
- **Passos para Reproduzir:** 1. Cadastrar curso Online com link. 2. Ver card na listagem.
- **Resultado Atual:** Link não aparece no card.
- **Resultado Esperado:** Link visível e clicável.
- **Impacto:** Dado essencial inacessível.

**BUG-08 — Endereço coletado mas não exibido no card**
- **Severidade:** Alta
- **Passos para Reproduzir:** 1. Cadastrar Presencial com endereço. 2. Ver card.
- **Resultado Atual:** Endereço não aparece.
- **Resultado Esperado:** Endereço visível no card.
- **Impacto:** Aluno não sabe onde ir.

**BUG-09 — Ausência de backend real gerando dados isolados**
- **Severidade:** Alta
- **Passos para Reproduzir:** 1. Cadastrar cursos no Chrome. 2. Abrir a mesma URL no Brave.
- **Resultado Atual:** Cursos do Chrome não aparecem no Brave.
- **Resultado Esperado:** Dados salvos em um servidor real.
- **Impacto:** Sistema sem banco de dados real, dados não são salvos definitivamente.

### Bugs Médios

**BUG-10 — Vagas decimais aceitas**
- **Severidade:** Média
- **Passos para Reproduzir:** 1. Preencher vagas com 2.5. 2. Submeter.
- **Resultado Atual:** Aceita o decimal.
- **Resultado Esperado:** Aceitar apenas números inteiros.
- **Impacto:** Dados sem sentido.

**BUG-11 — Datas no passado aceitas sem aviso**
- **Severidade:** Média
- **Passos para Reproduzir:** 1. Definir datas em 2020. 2. Submeter.
- **Resultado Atual:** Aceita normalmente.
- **Resultado Esperado:** Ao menos avisar o usuário.
- **Impacto:** Confusão entre cursos futuros e passados.

**BUG-12 — URL de imagem inválida aceita**
- **Severidade:** Média
- **Passos para Reproduzir:** 1. Preencher URL com texto-qualquer. 2. Submeter.
- **Resultado Atual:** Imagem quebrada no card.
- **Resultado Esperado:** Validação de URL ou exibir um placeholder.
- **Impacto:** Interface quebrada.

**BUG-13 — Header não responsivo**
- **Severidade:** Média
- **Passos para Reproduzir:** 1. Acessar em view 375px ou pegar um dispositivo móvel e abrir o site nele.
- **Resultado Atual:** Header quebra e elementos se sobrepõem.
- **Resultado Esperado:** Layout adaptado para mobile.
- **Impacto:** Inacessível em celular.

**BUG-14 — Nomes duplicados permitidos**
- **Severidade:** Média
- **Passos para Reproduzir:** 1. Cadastrar dois cursos com o mesmo nome.
- **Resultado Atual:** Ambos criados sem aviso.
- **Resultado Esperado:** Alertar sobre duplicidade.
- **Impacto:** Confusão ao gerenciar.

### Bugs Baixos

**BUG-15 — Dados do formulário perdidos ao recarregar a tela**
- **Severidade:** Baixa
- **Passos para Reproduzir:** 1. Preencher metade do form. 2. Ir para a listagem. 3. Voltar.
- **Resultado Atual:** Tudo limpo.
- **Resultado Esperado:** Preservar dados ou avisar antes de sair.
- **Impacto:** Retrabalho para o usuário.

---

## 6. Evidências da execução dos testes
Sendo cruciais para a validação real de tudo aquilo descrito neste relatório, as imagens ilustrativas (prints) e gravações focadas nos bugs e anomalias relatadas foram compiladas em uma pasta compartilhada na nuvem.

**Evidências dos testes:** [https://drive.google.com/drive/folders/1SDz70kVTIqx5vVarO3og54Zx9EddD6Bv?usp=drive_link]

---

## 7. Uso de Inteligência Artificial
A IA foi usada como uma assistente para os testes. Basicamente, eu anotava no meu caderno ou bloco de notas digital possíveis testes para ir fazendo, aqueles que considerava mais fáceis (como testar o resultado de deixar um curso sem informação x ou y), passava para ela e ela fazia de forma automatizada, abrindo o navegador sozinha e ia testando enquanto eu fazia testes mais complexos no meu outro navegador. Além disso, usei a IA para ajudar a fazer essa formatação em MD. O resto foram coisas minhas (como a planilha, montar o drive, pensar, fuçar os erros e montar o README).
