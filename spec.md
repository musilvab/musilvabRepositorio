## UC1 — Abrir bilhete

- **Endpoint:** `POST /bilhetes`
- **Ação:** Registrar a entrada de um veículo e criar um bilhete com status "aberto".
- **Entradas Esperadas (Payload):**
  - `placa` *(Obrigatório)*: string (7 caracteres alfanuméricos, maiúsculos).
  - `entrada` *(Opcional)*: string (Formato ISO-8601 com fuso `-03:00`).

### Critérios de Aceite
- **Dado** que recebo payload com placa válida (ex: `"ABC1D23"`) sem a chave `entrada`, **Quando** processo, **Então** o sistema cria o bilhete usando o tempo atual do servidor (injetado via dependência) e retorna status `201`.
- **Dado** que recebo payload com a chave `entrada` preenchida com data/hora válida, **Quando** processo, **Então** o bilhete é aberto com esse tempo exato fornecido, sobrescrevendo o relógio atual (gancho de testabilidade), e retorna status `201`.
- **Dado** que a placa é inválida ou ausente, **Quando** processo, **Então** retorno erro `422` com `{"erro": "placa_invalida"}`.
- **Dado** que a `entrada` está fora do formato ISO-8601 ou com fuso incorreto, **Quando** processo, **Então** retorno erro `422` com `{"erro": "entrada_invalida"}`.
- **Dado** que a placa informada já possui um bilhete "aberto", **Quando** processo, **Então** retorno erro `409` com `{"erro": "bilhete_em_aberto"}`.

---

## UC2 — Encerrar bilhete

- **Endpoint:** `POST /bilhetes/{id}/encerramento`
- **Ação:** Registra a saída do veículo, calcula o tempo de permanência e o valor a ser cobrado, alterando o status para "encerrado".

### Regras de Cobrança
- Cobra-se por fração de `FRACAO_MINUTOS`, sempre arredondando para cima (ex: passou 1 minuto da fração, cobra a próxima inteira).
- **Valor da fração:** `TARIFA_HORA_CENTAVOS` ÷ (60 ÷ `FRACAO_MINUTOS`).
- **Tolerância:** Os primeiros `TOLERANCIA_MINUTOS` são grátis (R$ 0). Se o tempo superar a tolerância, cobra-se desde o 1º minuto (a tolerância não é descontada do total).
- **Teto Diário:** O `valor_centavos` a ser pago nunca pode ultrapassar `TETO_DIARIO_CENTAVOS`.

### Critérios de Aceite
- **Dado** que o tempo de permanência do bilhete é menor ou igual a `TOLERANCIA_MINUTOS`, **Quando** solicito o encerramento, **Então** o sistema encerra o bilhete com `valor_centavos: 0` e retorna status `200`.
- **Dado** que o tempo de permanência ultrapassou a tolerância e consumiu 2 frações exatas e meia, **Quando** solicito o encerramento, **Então** o sistema arredonda para 3 frações, calcula o valor integral multiplicando pelo preço da fração e retorna status `200`.

---

## UC3 — Cancelar bilhete

- **Endpoint:** `POST /bilhetes/{id}/cancelamento`
- **Ação:** Cancela um bilhete utilizando o ID informado como parâmetro na URL.

### Critérios de Aceite
- **Dado** que o bilhete está com status "aberto", **Quando** solicito o cancelamento, **Então** o status do bilhete passa a ser "cancelado", retornando status `200` sem gerar horário de saída ou `valor_centavos` (sem cobrança).

---

## UC4 — Histórico por placa

- **Endpoint:** `GET /bilhetes?placa={placa}` (Ex: `?placa=ABC1D23`)
- **Ação:** Retorna o histórico de bilhetes usando a placa informada na URL.

### Critérios de Aceite
- **Dado** que a placa possui histórico, **Quando** consulto, **Então** o sistema retorna status `200` com a lista de todos os bilhetes.
- **Dado** que a placa é válida mas não tem bilhetes registrados, **Quando** consulto, **Então** o sistema retorna status `200` informando que a placa não tem reservas.
- **Dado** que a placa informada é inválida, **Quando** consulto, **Então** o sistema retorna erro `404`, informando que a placa não foi encontrada.

---

## UC5 — Relatório Diário

- **Endpoint:** `GET /relatorios/diario?data=AAAA-MM-DD`
- **Ação:** Retorna um relatório consolidado referente ao dia especificado.

### Regras do Relatório
- O relatório deve informar: `data`, `total de bilhetes`, `faturamento` em centavos, e o `tempo médio` em minutos.
- O tempo médio deve considerar apenas os bilhetes **encerrados** no dia, aplicando arredondamento de 0.5 para cima.

### Critérios de Aceite
- **Dado** que existem registros no dia, **Quando** consulto o relatório, **Então** o sistema retorna os dados consolidados.
- **Dado** que não há dados no dia, **Quando** consulto o relatório, **Então** o sistema retorna um erro informando que não há lista até o presente momento do dia.

---

## Regras de Negócio Globais

- **Tolerância Gratuita:** Os primeiros `TOLERANCIA_MINUTOS` de um bilhete são grátis (duração ≤ tolerância → `valor_centavos: 0`). Se o veículo passar da tolerância (mesmo por 1 minuto), cobra-se de forma integral desde o primeiro minuto — a tolerância não é descontada do tempo total.
- **Uma Vaga por Placa:** Ao realizar um `POST /bilhetes` para uma placa que já tem um bilhete "aberto", o sistema deve barrar e retornar `409` com `{"erro": "bilhete_em_aberto"}`. A placa só volta a poder abrir um novo bilhete após o anterior ser encerrado ou cancelado.