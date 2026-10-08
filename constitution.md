# Constitution

1. Será desenvolvida uma API para operadora de estacionamento rotativo.
2. Código e identificadores precisam estar em inglês, e sua documentação em português, seguindo o padrão de `snake_case`.
3. A aplicação será **contêiner first**.
4. > [!IMPORTANT]  
   > A variável de ambiente será `PORTA_SERVICO`. A aplicação precisa rodar em um contêiner Docker e deve expor e escutar a porta definida pela variável de ambiente `PORTA_SERVICO`. Não colocar valores fixos no código (*hardcoded*).
5. Variáveis como `TARIFA_HORA_CENTAVOS`, como diz o nome, o cálculo será em centavos, e será calculado o horário cheio. A tolerância de minutos iniciais por bilhete é de 0, 10 ou 15 minutos; passou da tolerância, cobra-se desde o primeiro minuto.
6. A formatação de horário usada será a **ISO 8601** com o fuso horário de `-03:00`.