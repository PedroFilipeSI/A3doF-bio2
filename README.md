# Sistema de Gestão de Clínica Veterinária & Integração Governamental

Sistema completo (Full Stack) para gestão de clínica veterinária com integração governamental para rastreabilidade de microchips RFID e notificação obrigatória de vacinação antirrábica.

### Arquitetura do Sistema
1. **Frontend (Interface Web):** Aplicação web para recepção e médicos veterinários operarem o sistema (login, tutores, animais, microchips, consultas e vacinas).
2. **`api_clinica` (Backend Principal — Porta `8000`):** API REST em FastAPI conectada ao **PostgreSQL** (`clinica_db`), responsável pelas regras de negócio, autenticação JWT, consulta de CEP via **ViaCEP** e disparo assíncrono/resiliente para o governo.
3. **`api_governamental` (Serviço Externo Simulado — Porta `8001`):** API REST em FastAPI conectada ao **SQLite** (`governo.db`), simulando a base nacional de animais microchipados e vacinas antirrábicas com controle de **idempotência** (`Idempotency-Key`).

--

## Estrutura do Repositório

```text
Clinica-veterinaria/
├── backend/
│   ├── api_clinica/              # API da Clínica (Porta 8000)
│   │   ├── main.py               # Endpoints e regras de integração
│   │   ├── database.py           # Conexão SQLAlchemy (PostgreSQL)
│   │   ├── models.py             # Modelos das tabelas do clinica_db
│   │   ├── schemas.py            # Validação Pydantic
│   │   ├── security.py           # Autenticação JWT e hash bcrypt
│   │   ├── requirements.txt      # Dependências Python
│   │   └── .env                  # Configurações locais (criar na instalação)
│   │
│   └── api_governamental/        # API Governamental (Porta 8001)
│       ├── main.py               # Endpoints governamentais e idempotência
│       └── governo.db            # Banco SQLite criado automaticamente
│
├── frontend/                     # Interface Web (Site da Clínica)
├── database/                     # Scripts SQL (DDL e carga inicial)
└── documentacao/
    └── colecao_postman.json      # Coleção Postman validada (16 cenários)

Guia de Instalação:

1. Pré-requisitos na Máquina
Certifique-se de que o computador possui instalados:
• Git
• Pthon 3.11+
• PostgreSQL (rodando na porta 5432) + DBEAVER

2. Clonar o Repositório
Abra o terminal (PowerShell ou CMD) e baixe o projeto:
git clone <URL_DO_REPOSITORIO>
cd Clinica-veterinaria

3. Configurar o Banco de Dados:

1- Abra o DBeaver, conecte no PostgreSQL local e crie um banco chamado clinica_db.

2- Execute o script SQL de criação das tabelas (localizado na pasta database/ ou raiz do projeto) para criar:usuarios, veterinarios, enderecos, tutores, especies, racas, animais, microchips, consultas, vacinas e integracoes_governo.

3- Carga Inicial Obrigatória (Seed): Insira pelo menos 1 registro nas tabelas base para permitir cadastros e login imediato:
INSERT INTO especies (id, nome) VALUES (1, 'Canina'), (2, 'Felina');
INSERT INTO racas (id, especie_id, nome) VALUES (1, 1, 'Golden Retriever'), (2, 1, 'SRD'), (3, 2, 'Siamês');



4. Configurar o Backend:

1- Entre na pasta da API da clínica, crie o ambiente virtual e instale as dependências:

cd backend\api_clinica
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt

2- Crie o arquivo .env dentro de backend/api_clinica/ (use o .env.example como base, se houver) ajustando a senha do PostgreSQL da máquina atual:

DATABASE_URL=postgresql+psycopg://postgres:SENHA_DO_POSTGRES@localhost:5432/clinica_db
SECRET_KEY=chave_secreta_a3_clinica_vet_2026
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
GOVERNO_API_URL=[http://127.0.0.1:8001](http://127.0.0.1:8001)
GOVERNO_API_KEY=gov-api-key-secreta-2026

5. Subir os Servidores:

Terminal 1:
cd Clinica-veterinaria\backend\api_clinica
.\venv\Scripts\python.exe -m uvicorn main:app --port 8000 --reload

Teste rápido no navegador: http://127.0.0.1:8000/docs

Terminal 2:
cd Clinica-veterinaria\backend\api_governamental
..\api_clinica\venv\Scripts\python.exe -m uvicorn main:app --port 8001 --reload

Teste rápido no navegador: http://127.0.0.1:8001/docs

Terminal 3:

cd Clinica-veterinaria\frontend
npm install
npm run dev
