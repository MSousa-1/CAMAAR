# CAMAAR

Sistema para avaliação de atividades acadêmicas remotas do CIC 

Sendo desenvolvido por: Marcus Vinicius Ramalho de Sousa

# Especificação do Projeto e Rota de Desenvolvimento

## 1. Visão Geral
O **CAMAAR** é um sistema de gerenciamento de formulários onde o administrador importa dados de alunos e cria formulários para avaliação de turmas. O sistema utiliza dados do SIGAA para popular sua base e permitir o acesso dos participantes.

---

## 2. Rota de Desenvolvimento (Roadmap)

O desenvolvimento foi organizado em **5 Etapas** para garantir que as dependências (Dados -> Templates -> Formulários) sejam respeitadas.

**Velocity Total: 80 Pontos**

### Etapa 1: Autenticação e Gestão de Acesso
*Foco: Garantir que usuários entrem no sistema e tenham a visualização correta (Admin vs. Usuário).*

| ID | História de Usuário | Responsável | Velocity |
| :--- | :--- | :--- | :--- |
| [**US-104**](https://github.com/EngSwCIC/CAMAAR/issues/104) | Eu como **Usuário do sistema**<br>Quero acessar o sistema utilizando um e-mail ou matrícula e uma senha já cadastrada<br>A fim de responder formulários ou gerenciar o sistema<br>_Obs: Quando o Usuário logado for um admin, deve-se mostrar a opção de gerenciamente no menu lateral._ | Desenvolvedor | **5** |
| [**US-105**](https://github.com/EngSwCIC/CAMAAR/issues/105) | Eu como **Usuário**<br>Quero definir uma senha para o meu usuário a partir do e-mail do sistema de solicitação de cadastro<br>A fim de acessar o sistema  | Desenvolvedor | **5** |
| **Infra** | Configuração inicial do Banco de Dados e Repositório | Desenvolvedor | **3** |

**Total: 13 Pontos**

#### Regras de Negócio
* **RN-01 (Login):** O sistema deve validar se o login é um e-mail válido ou um número de matrícula existente.
* **RN-02 (Sessão):** Deve haver distinção de Aluno e Admin
* **RN-03 (Senha):** A senha definida pelo usuário deve ter no mínimo 8 caracteres.
---

### Etapa 2: Ingestão de Dados (SIGAA)
*Foco: Alimentar a base de dados.*

| ID | História de Usuário | Responsável | Velocity |
| :--- | :--- | :--- | :--- |
| [**US-98**](https://github.com/EngSwCIC/CAMAAR/issues/98) | Eu como **Administrador**<br>Quero importar dados de turmas, matérias e participantes do SIGAA (caso não existam na base de dados atual)<br>A fim de alimentar a base de dados do sistema.<br>_P.S. utilizar os JSONs presentes no repositório_ | Desenvolvedor | **8** |
| [**US-100**](https://github.com/EngSwCIC/CAMAAR/issues/100) | Eu como **Administrador**<br>Quero cadastrar participantes de turmas do SIGAA ao importar dados de usuarios novos para o sistema<br>A fim de que eles acessem o sistema CAMAAR | Desenvolvedor | **5** |
| [**US-108**](https://github.com/EngSwCIC/CAMAAR/issues/108) | Eu como **Administrador**<br>Quero atualizar a base de dados já existente com os dados atuais do sigaa<br>A fim de corrigir a base de dados do sistema. | Desenvolvedor | **8** |

**Total: 21 Pontos**

#### Regras de Negócio
* **RN-04 (Unicidade):** O sistema não pode criar duplicatas. A validação deve ser feita pelo campo `matrícula` (para alunos) e `código da turma` + `semestre` (para turmas).
* **RN-05 (Vínculo):** Ao importar, o sistema deve criar automaticamente o relacionamento na tabela associativa `matriculas` (Aluno <-> Turma).
* **RN-06 (Integridade):** Se o arquivo JSON contiver erros de formatação, a importação deve ser totalmente abortada (All-or-nothing) e um erro reportado.
* **RN-07 (Atualização):** Se um aluno já existe mas mudou de turma no novo JSON, o sistema deve apenas atualizar o vínculo, mantendo o histórico e os dados de login do aluno intactos.

---

### Etapa 3: Gestão de Templates
*Foco: Criação e manutenção dos modelos de perguntas.*

| ID | História de Usuário | Responsável | Velocity |
| :--- | :--- | :--- | :--- |
| [**US-102**](https://github.com/EngSwCIC/CAMAAR/issues/102) | Eu como **Administrador**<br>Quero criar um template de formulário contendo as questões do formulário<br>A fim de gerar formulários de avaliações para avaliar o desempenho das turmas | Desenvolvedor | **8** |
| [**US-111**](https://github.com/EngSwCIC/CAMAAR/issues/111) | Eu como **Administrador**<br>Quero visualizar os templates criados<br>A fim de poder editar e/ou deletar um template que eu criei | Desenvolvedor | **2** |
| [**US-112**](https://github.com/EngSwCIC/CAMAAR/issues/112) | Eu como **Administrador**<br>Quero editar e/ou deletar um template que eu criei sem afetar os formulários já criados<br>A fim de organizar os templates existentes | Desenvolvedor | **5** |

**Total: 15 Pontos**

#### Regras de Negócio
* **RN-08 (Conteúdo Mínimo):** Um template não pode ser salvo sem um Título e pelo menos uma Questão.
* **RN-09 (Imutabilidade Histórica):** Se um template já foi utilizado para gerar um Formulário que recebeu respostas, ele **não pode** ser editado diretamente.
    * *Solução:* O sistema deve bloquear a edição ou sugerir "Salvar como Novo Template".
* **RN-10 (Soft Delete):** A exclusão de um template deve ser lógica (`is_active = false`), para não quebrar relatórios passados que referenciam este template.

---

### Etapa 4: Criação e Distribuição de Formulários
*Foco: Transformar templates em formulários ativos para as turmas e permitir a visualização pelo aluno.*

| ID | História de Usuário | Responsável | Velocity |
| :--- | :--- | :--- | :--- |
| [**US-103**](https://github.com/EngSwCIC/CAMAAR/issues/103) | Eu como **Administrador**<br>Quero criar um formulário baseado em um template para as turmas que eu escolher<br>A fim de avaliar o desempenho das turmas no semestre atual | Desenvolvedor | **8** |
| [**US-109**](https://github.com/EngSwCIC/CAMAAR/issues/109) | Eu como **Participante de uma turma**<br>Quero visualizar os formulários não respondidos das turmas em que estou matriculado<br>A fim de poder escolher qual irei responder | Desenvolvedor | **5** |
| **UI** | Implementação de filtros de busca por formulário/turma (Baseado no layout) | Desenvolvedor | **2** |

**Total: 15 Pontos**

#### Regras de Negócio
* **RN-11 (Prazo):** Todo formulário deve ter uma Data de Início e uma Data de Fim. O sistema só deve mostrá-lo aos alunos dentro desse período.
* **RN-12 (Cópia de Estrutura):** Ao criar um formulário, as perguntas do Template devem ser copiadas para a instância do Formulário. Isso garante que alterações futuras no Template não mudem formulários já lançados.
* **RN-13 (Visibilidade):** O usuário só pode visualizar formulários vinculados às turmas nas quais sua matrícula está ativa no banco de dados.

---

### Etapa 5: Respostas e Resultados
*Foco: O aluno responde e o administrador extrai os dados.*

| ID | História de Usuário | Responsável | Velocity |
| :--- | :--- | :--- | :--- |
| [**US-99**](https://github.com/EngSwCIC/CAMAAR/issues/99) | Eu como **Participante de uma turma**<br>Quero responder o questionário sobre a turma em que estou matriculado<br>A fim de submeter minha avaliação da turma | Desenvolvedor | **8** |
| [**US-110**](https://github.com/EngSwCIC/CAMAAR/issues/110) | Eu como **Administrador**<br>Quero visualizar os formulários criados<br>A fim de poder gerar um relatório a partir das respostas | Desenvolvedor | **3** |
| [**US-101**](https://github.com/EngSwCIC/CAMAAR/issues/101) | Eu como **Administrador**<br>Quero baixar um arquivo csv contendo os resultados de um formulário<br>A fim de avaliar o desempenho das turmas | Desenvolvedor | **5** |

**Total: 16 Pontos**

#### Regras de Negócio
* **RN-14 (Resposta Única):** Um aluno só pode submeter resposta para um formulário específico uma única vez. O sistema deve bloquear tentativas subsequentes.
* **RN-15 (Obrigatoriedade):** O formulário não pode ser enviado se houver questões marcadas como "Obrigatória" sem resposta.
* **RN-16 (Privacidade):** O relatório CSV deve exportar os dados brutos das respostas.
* **RN-17 (Status):** O formulário deve mudar de status automaticamente para "Fechado" assim que a data atual superar a Data Fim configurada.

---

## 3. Modelagem de Dados (Conceitual)

Diagrama simplificado das entidades principais para suportar o fluxo de Templates Dinâmicos e Importação.
![Diagrama ER](https://github.com/MSousa-1/CAMAAR/blob/Sprint-1/DiagramaER.png)

---

## 4. Cenários de Teste (BDD)

### [Etapa 1 - Autenticação e Gestão de Acesso](https://github.com/MSousa-1/CAMAAR/Sprint-1/README.md#etapa-1-autentica%C3%A7%C3%A3o-e-gest%C3%A3o-de-acesso)

| ID | História de Usuário | Qtd. Testes | Detalhamento dos Cenários (BDD) |
| :--- | :--- | :---: | :--- |
| [**US-01**](https://github.com/MSousa-1/CAMAAR/edit/Sprint-1/README.md) | Login | **4** | 1. Sucesso (Admin).<br>2. Sucesso (Aluno/Participante).<br>3. Falha (Dados inválidos).<br>4. Falha (Campos vazios ou formato de e-mail inválido). |
| [**US-105**](https://github.com/EngSwCIC/CAMAAR/issues/105) | Definição de Senha | **3** | 1. Sucesso (Senha forte definida).<br>2. Falha (Senha < 8 caracteres).<br>3. Falha (Senhas de confirmação não conferem). |

### [Etapa 2: Ingestão de Dados (SIGAA)](https://github.com/MSousa-1/CAMAAR/edit/Sprint-1/README.md#etapa-2-ingest%C3%A3o-de-dados-sigaa)

| ID | História de Usuário | Qtd. Testes | Detalhamento dos Cenários (BDD) |
| :--- | :--- | :---: | :--- |
| [**US-98**](https://github.com/EngSwCIC/CAMAAR/issues/98) | Importar SIGAA | **5** | 1. Sucesso (Arquivo JSON Perfeito).<br>2. Falha (Arquivo corrompido ou extensão incorreta).<br>3. Falha (Estrutura do JSON incorreta - campos obrigatórios faltando).<br>4. Falha (Arquivo excedendo limite de tamanho).<br>5. Validação (Feedback visual de progresso/conclusão). |
| [**US-100**](https://github.com/EngSwCIC/CAMAAR/issues/100) | Cadastrar Usuários | **3** | 1. Sucesso (Criação de novos usuários).<br>2. Ignorar (Usuários sem e-mail ou matrícula válidos).<br>3. Log de erro (Relatório de registros que falharam). |
| [**US-108**](https://github.com/EngSwCIC/CAMAAR/issues/108) | Atualizar Base | **4** | 1. Sucesso (Atualizar vínculo de turma de aluno existente).<br>2. Sucesso (Não duplicar registro de usuário já existente).<br>3. Falha (Conflito de ID/Matrícula duplicada no mesmo arquivo).<br>4. Integridade (Aluno removido do JSON original mantém histórico antigo). |

### [Etapa 3: Gestão de Templates](https://github.com/MSousa-1/CAMAAR/edit/Sprint-1/README.md#etapa-3-gest%C3%A3o-de-templates)

| ID | História de Usuário | Qtd. Testes | Detalhamento dos Cenários (BDD) |
| :--- | :--- | :---: | :--- |
| [**US-102**](https://github.com/EngSwCIC/CAMAAR/issues/102) | Criar Template | **3** | 1. Sucesso (Template completo com perguntas).<br>2. Falha (Tentar salvar sem Título).<br>3. Falha (Tentar salvar sem nenhuma pergunta adicionada). |
| [**US-111**](https://github.com/EngSwCIC/CAMAAR/issues/111) | Visualizar Templates | **2** | 1. Visualização (Lista carregada corretamente).<br>2. Estado Vazio (Feedback visual quando não há templates). |
| [**US-112**](https://github.com/EngSwCIC/CAMAAR/issues/112) | Editar/Deletar | **4** | 1. Sucesso (Editar template sem uso).<br>2. Sucesso (Deletar template sem uso).<br>3. Bloqueio (Tentar editar template já vinculado a formulário respondido).<br>4. Bloqueio (Tentar deletar template já vinculado a histórico). |

### [Etapa 4: Criação e Distribuição de Formulários](https://github.com/MSousa-1/CAMAAR/edit/Sprint-1/README.md#etapa-4-cria%C3%A7%C3%A3o-e-distribui%C3%A7%C3%A3o-de-formul%C3%A1rios)

| ID | História de Usuário | Qtd. Testes | Detalhamento dos Cenários (BDD) |
| :--- | :--- | :---: | :--- |
| [**US-103**](https://github.com/EngSwCIC/CAMAAR/issues/103) | Criar Formulário | **4** | 1. Sucesso (Vínculo Turma + Template criado).<br>2. Falha (Data de Término anterior à Data de Início).<br>3. Falha (Datas no passado).<br>4. Validação (Selecionar turma vazia/sem alunos). |
| [**US-109**](https://github.com/EngSwCIC/CAMAAR/issues/109) | Ver Pendências | **3** | 1. Sucesso (Visualizar formulário dentro do prazo).<br>2. Ocultação (Não visualizar formulário expirado).<br>3. Ocultação (Não visualizar formulário já respondido pelo aluno). |

### [Etapa 5: Respostas e Resultados](https://github.com/MSousa-1/CAMAAR/edit/Sprint-1/README.md#etapa-5-respostas-e-resultados)

| ID | História de Usuário | Qtd. Testes | Detalhamento dos Cenários (BDD) |
| :--- | :--- | :---: | :--- |
| [**US-99**](https://github.com/EngSwCIC/CAMAAR/issues/99) | Responder | **5** | 1. Sucesso (Fluxo completo de resposta).<br>2. Falha (Tentar enviar com obrigatórias em branco).<br>3. Segurança (Idempotência - Bloquear envio duplicado).<br>4. Segurança (Tentar alterar respostas via API após envio).<br>5. UI (Feedback de sucesso após submissão). |
| [**US-110**](https://github.com/EngSwCIC/CAMAAR/issues/110) | Status | **2** | 1. Precisão (Contagem correta de respondentes vs total).<br>2. Status correto (Aberto/Fechado baseado na data). |
| [**US-101**](https://github.com/EngSwCIC/CAMAAR/issues/101) | Exportar CSV | **3** | 1. Sucesso (Download e validação das colunas/linhas).<br>2. Tratamento (Sanitização de caracteres especiais no texto).<br>3. Estado Vazio (Tentativa de baixar CSV de formulário sem respostas). |

---
