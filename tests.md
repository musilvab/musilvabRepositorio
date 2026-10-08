# Tests (tests.md)

> [!TIP]
> Os testes de negócio devem validar o comportamento exato das frações, do teto e da tolerância. O Pytest deve simular as variáveis de ambiente utilizando os seguintes valores para rodar as asserções: `TOLERANCIA_MINUTOS=10`, `FRACAO_MINUTOS=30`, `TARIFA_HORA_CENTAVOS=500` (logo, 250 por fração), `TETO_DIARIO_CENTAVOS=5000`.

## 1. Tabela de Casos de Borda do Tempo de Permanência (UC2)

| Cenário de Borda | Tempo de Permanência | Valor Cobrado Esperado | Motivo / Regra Testada |
| :--- | :--- | :--- | :--- |
| Exato limite da tolerância | 10 min | 0 | Está dentro da regra de tolerância grátis. |
| Adjacência (+1 min) | 11 min | 250 | Passou a tolerância, cobra integral desde o minuto zero (1 fração cobrada). |
| Limite da primeira fração | 30 min | 250 | Tempo exato preenche 1 fração de 30 min. |
| Passou 1 min da fração | 31 min | 500 | Arredondamento para cima: cobra 2 frações de 30 min. |
| Limite antes do Teto | 599 min | 5000 | 20 frações consumidas atingem exatos 5000 centavos. |
| Rompeu o Teto Diário | 800 min | 5000 | O valor calculado seria 6750, mas o teto trava a cobrança em 5000. |

## 2. Testes de Arredondamento Bancário (UC4)
- **Cenário:** Tempo médio no relatório calculando 47.5 minutos.
- **Esperado:** O valor retornado deve ser 48 minutos, seguindo a regra estrita do contrato (arredondar 0,5 para cima).