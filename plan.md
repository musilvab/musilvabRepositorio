# Plan

## 1. Stack Tecnológica
Python 3.11, FastAPI, Uvicorn e Pytest. É proibido adicionar outras bibliotecas de terceiros.

## 2. Decisões de Arquitetura e Justificativas
- **FastAPI:** Escolhido porque a validação nativa via Pydantic garante os retornos de erro 422 (placas com formato incorreto, datas inválidas).
- **Uvicorn:** Atua como o servidor ASGI para receber as requisições HTTP.
- **Pytest:** Escolhido por sua arquitetura, para simular a passagem do tempo para testes.
- **Persistência:** Em memória (uso de dicionários/listas).

## 3. Injeção de Dependência (Relógio)
A arquitetura deve isolar a captura da data e hora atual em uma dependência injetável para que o Pytest consiga injetar horários falsos simulando horas passadas, facilitando a testabilidade de tolerância e teto diário exigidos.

## 4. Estratégia de Conteinerização (Docker)
- A aplicação será empacotada com um `Dockerfile` simples e as dependências centralizadas no `requirements.txt`.
- O comando de inicialização (`CMD` ou `ENTRYPOINT`) precisa chamar o Uvicorn (`uvicorn main:app --host 0.0.0.0`), mapeando a porta dinamicamente a partir da variável de ambiente `PORTA_SERVICO`.