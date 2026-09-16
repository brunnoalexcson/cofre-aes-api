"# cofre-aes-api" 
"# cofre-aes-api" 

# Cofre de Senhas Corporativo

Projeto da disciplina Criptografia Aplicada — PUC Goiás.

## Integrantes

* Brunno Alexcson de Carvalho
* Caio Almeida Oliveira
* Daniel Junio Barbosa Souza Filho 

## Objetivo

API para armazenar credenciais no PostgreSQL hospedado no Supabase.
As senhas são protegidas com AES-256-GCM, utilizando uma chave derivada
da senha-mestra por PBKDF2-HMAC-SHA256.

Título, usuário e URL permanecem em texto legível. A senha-mestra e a
chave derivada não são persistidas nem mantidas entre requisições.

## Instalação e configuração

Clone o repositório e abra um terminal na pasta do projeto.

Crie o ambiente virtual:

powershell
python -m venv .venv


Ative no PowerShell:

powershell
.\.venv\Scripts\Activate.ps1


Instale as dependências:

powershell
python -m pip install -r requirements.txt


Crie um projeto próprio da equipe no Supabase e execute o conteúdo de
sql/esquema.sql no SQL Editor.

Copie .env.exemplo para .env:

powershell
Copy-Item .env.exemplo .env


Preencha .env com a URL e a chave pública do projeto:

dotenv
SUPABASE_URL=URL_DO_PROJETO
SUPABASE_KEY=CHAVE_PUBLICA_DO_PROJETO


O arquivo .env não deve ser enviado ao GitHub.

## Execução

Na raiz do projeto, com o ambiente virtual ativo:

powershell
python -m uvicorn app.main:app --reload


Acesse a interface de testes:

http://127.0.0.1:8000/docs

## Organização do código

* app/cripto.py: derivação de chave, cifragem, decifragem e verificador.
* app/banco.py: operações de acesso ao Supabase.
* app/modelos.py: modelos de entrada e saída.
* app/main.py: rotas HTTP e integração entre os módulos.
* sql/esquema.sql: criação das tabelas e políticas.
* testes/resultados.md: resultados dos testes.
* testes/evidencias/: evidências das verificações.

## Rotas

Com exceção de POST /cofres, todas as rotas exigem o cabeçalho
X-Senha-Mestra.

| Método | Rota | Finalidade |
|---|---|---|
| POST | /cofres | Criar um cofre |
| POST | /cofres/{id}/abrir | Verificar a senha-mestra |
| POST | /cofres/{id}/segredos | Cadastrar uma credencial |
| GET | /cofres/{id}/segredos | Listar metadados, sem senhas |
| GET | /cofres/{id}/segredos/{sid} | Recuperar uma senha |
| PUT | /cofres/{id}/segredos/{sid} | Atualizar a senha com novo nonce |
| DELETE | /cofres/{id}/segredos/{sid} | Excluir uma credencial |

### Exemplo de criação de cofre

Requisição: POST /cofres

json
{
  "nome": "Cofre de testes",
  "senha_mestra": "<senha-mestra escolhida>"
}


Resposta esperada: HTTP 201

json
{
  "id": "<UUID gerado>"
}


### Exemplo de cadastro de segredo

Requisição: POST /cofres/{id}/segredos

Cabeçalho: X-Senha-Mestra: <senha-mestra escolhida>

json
{
  "titulo": "Servico de testes",
  "usuario": "usuario-teste",
  "url": "https://example.com",
  "senha": "<senha ficticia para o teste>"
}


Resposta esperada: HTTP 201

json
{
  "id": "<UUID do segredo>"
}

### Respostas HTTP

* 200: operação concluída.
* 201: cofre ou segredo criado.
* 401: falha na verificação da senha-mestra.
* 404: cofre ou segredo inexistente.
* 422: requisição inválida.
* 500: falha de integridade do segredo após autenticação do cofre,
  ou parâmetros inválidos do cofre.
* 503: falha tratada de configuração ou acesso ao banco.

## Parâmetros criptográficos

A chave é derivada com PBKDF2-HMAC-SHA256, utilizando 210.000 iterações,
sal aleatório de 16 bytes por cofre e saída de 32 bytes.

As iterações aumentam o custo de cada tentativa de senha. O sal
individualiza a derivação entre cofres. A saída de 32 bytes fornece
a chave de 256 bits utilizada pelo AES.

A cifragem utiliza AES-256-GCM com nonce aleatório de 12 bytes por
operação, inclusive nas atualizações, e etiqueta de autenticação de
16 bytes. O GCM permite verificar a integridade do conteúdo cifrado.

O AAD dos segredos é cofre_id|segredo_id, codificado em UTF-8.
Isso vincula o conteúdo cifrado ao registro e impede que a cópia dos
campos criptográficos para outro registro seja aceita na leitura.

O verificador cifra a frase cofre-ok e utiliza apenas o identificador
do cofre como AAD. Sua verificação permite distinguir a falha da
senha-mestra da adulteração de um segredo.

Os campos binários são armazenados em Base64. Base64 apenas representa
bytes em texto; a proteção criptográfica é fornecida pelo AES-GCM.

## Limitações

* Título, usuário e URL permanecem legíveis no banco.
* Senhas-mestras fracas continuam vulneráveis a tentativas de adivinhação;
  o PBKDF2 aumenta o custo dessas tentativas.
* A API recebe a senha-mestra e processa senhas em memória. O projeto
  não protege contra um servidor comprometido.
* O uso fora do ambiente local exige HTTPS para proteger o tráfego.
* As políticas amplas do Supabase são didáticas. Quem possui a chave
  pública pode acessar as tabelas conforme essas políticas.
* A cifragem não impede exclusão de registros ou indisponibilidade.
* Não há recuperação de senha-mestra, controle individual de usuários
  ou sessão que mantenha a chave entre requisições.