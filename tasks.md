# Tasks (tasks.md)

1. **Configurar Infraestrutura e Dependências (Docker First):**
   - Criar o `requirements.txt` com as dependências do FastAPI, Uvicorn e Pytest.
   - Criar o `Dockerfile` que exponha e utilize a variável de ambiente `$PORTA_SERVICO` no comando de inicialização do servidor Uvicorn.

2. **Criar a Estrutura Base e Injeção de Dependência:**
   - Criar o `main.py` e organizar a arquitetura em `models.py` (para os schemas Pydantic) e `services.py` (para as regras de negócio em memória).
   - Implementar o Provider de Tempo injetável para permitir mock de datas.

3. **Desenvolver os Endpoints (UCs):**
   - Implementar os endpoints respeitando rigorosamente os payloads em `snake_case` e os status HTTP de erro (`422`, `404`, `409`) previstos no `spec.md`.

4. **Escrever a Suíte de Testes (Pytest):**
   - Implementar todos os casos de teste da tabela de borda (`tests.md`), utilizando a injeção do relógio para validar a cobrança precisa de frações, tolerância e teto diário.