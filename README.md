# Beeteller - Desenvolvedor Back-end

O objetivo dessa atividade é avaliar tecnicamente os candidatos que participam da nossa seleção para vaga de desenvolver Back-end. O teste é realizado para as vagas de todos os níveis de senioridade, mas para cada nível existe critérios mais específicos. 
Preste bastante atenção nas instruções e boa sorte! :)


## Instruções

Você deverá criar um projeto público no github e desenvolver todo o seu código dentro desse repositório. Use o *README* do seu repositório para explicar um pouco de como foi o desafio, as decisões que você tomou e as instruções para instalar e rodar corretamente o projeto.

Sinta-se livre e tente mostrar a sua capacidade nos impressionando, mas não esqueça de atingir os objetivos principais do projeto. Faça o seu melhor!

## Let's code

Você irá construir uma aplicação back-end que será utilizado para coleta de mensagens Pix. A aplicação deverá implementar um mecanismo avançado para gerenciamento de mensagens com alto volume e escala de acordo com os requisitos detalhados nas próximas seções.

O detalhamento abaixo tem como base a implementação utilizada pela Interface de Comunicação do SPI (Banco Central), para prover o Pix.


### Glossário

* ISPB: Código de 8 dígitos que identifica uma instituição de pagamento no Sistema de Pagamento Brasileiro (SPB). Como exemplo pode utilizar: "32074986".

### Endpoints

#### GET `/api/pix/{ispb}/stream/start`
Para iniciar a recuperação de mensagens Pix recebidas por uma instituição (utilizando o `ispb` como chave), deverá realizar um GET para esse endpoint. O PSP (instituição) poderá escolher se deseja receber uma mensagem por vez ou várias mensagens na mesma requisição. A escolha é feita
através do cabeçalho `Accept`. Quando o cabeçalho for inexistente ou seu valor for `application/json`, a API retornará apenas uma mensagem. Quando seu valor for `multipart/json`, poderá retornar múltiplas mensagens em uma mesma resposta. 

A API retorna no máximo 10 mensagens a cada requisição com `multipart`.

A API deverá retornar apenas as mensagens PIX em que o `ispb` utilizado no parâmetro seja o mesmo do `recebedor` da mensagem Pix.

Quando houver resultados, o `status code` esperado é 200. Quando não houver, o `status code` esperado é 204.

No cabeçalho de resposta deve retornar um cabeçalho `Pull-Next` com a URI que deve ser utilizado para a recuperação das próximas mensagens. Exemplo:
* `/api/pix/11111111/stream/5oj7tm0jow61` onde o "5oj7tm0jow61" é o `interationId`.

O PSP irá utilizar essa URI para buscar as próximas mensagens, de forma recursiva até que deseja interromper o processo de leitura.

Quando desejar interromper o processo de leitura, o PSP irá realizar uma chamada `DELETE` utilizando a URI fornecida no cabeçalho `Pull-Next` da última interação realizada.

#### GET `/api/pix/{ispb}/stream/{interationId}`
Esse endpoint terá o mesmo comportamento do `stream/start`, com o mesmo retorno de dados e cabeçalhos. Será uma chamada de continuação ao stream iniciado anteriormente.


#### DELETE `/api/pix/{ispb}/stream/{interationId}`
A chamada utilizando esse verbo é importante para sinalizar o recebimento com sucesso das mensagens anteriormente e liberar o stream para outros coletores (se necessário).

Após interromper o processo de leitura, o PSP só poderá iniciar o processo de coleta novamente utilizando o endpoint `/api/pix/{ispb}/stream/start`.

_____________________________

#### Long Polling

Em qualquer tentativa de leitura (GET), a API não deverá responder imediatamente caso não haja nenhuma mensagem disponível ao PSP. Ao invés disso, aguarda por alguns segundos até que uma mensagem esteja disponível.

Após o tempo decorrido sem nenhuma mensagem ao PSP, a API retornará um código 204 (No Content). A resposta 204 retorna, também, um cabeçalho `Pull-Next` que deve ser usado na próxima requisição. O tempo máximo esperado deve ser de 8 segundos.

O `Pull-Next` de um 204 pode ser idêntico ao retornado anteriormente, a critério da API. É importante que o PSP sempre obedeça ao que vier nesse cabeçalho. No caso de leituras que admitam mais de uma mensagem na resposta (multipart), a API responde ao PSP assim que houver ao menos uma mensagem disponível (200) ou após o tempo máximo de espera (204). A API não aguarda, portanto, por acúmulo de mensagens antes da resposta. Dessa forma, caso houver baixo volume de mensagens, uma parte das respostas com conteúdo carregará apenas uma mensagem, mesmo
em formato multipart.

#### Leitura simultânea

A API deverá permitir o consumo de forma paralela das mensagens por vários "coletores" atráves do GET `/api/pix/{ispb}/stream/start`. Dessa forma, cada requisição ao recurso stream/start pode fazer a API criar um “stream” de leitura.

A API deverá permitir no máximo 6 coletores simultâneamente. Quando atingir o limite, as requisições ao recurso stream/start devem retornar o erro com `status code` 429. O stream é considerado finalizado quando ocorre a chamada com DELETE ao recurso, conforme detalhado anteriormente.

A API deverá tratar e garantir que as mensagens não apareçam em mais de um stream, para evitar transações duplicadas. 


#### Dados esperados

Os dados esperados no endpoint de consulta (consulta de mensagens Pix) devem ser baseados no JSON seguinte:
```json
{
    "endToEndId": "E320749862024022119277T3lEBbUM0z",
    "valor": 90.20,
    "pagador": {
        "nome": "Marcos José",
        "cpfCnpj": "98716278190",
        "ispb": "32074986",
        "agencia": "0001",
        "contaTransacional": "1231231",
        "tipoConta": "CACC"
    },
    "recebedor": {
        "nome": "Flavio José",
        "cpfCnpj": "77615678291",
        "ispb": "00000000",
        "agencia": "0361",
        "contaTransacional": "1210098",
        "tipoConta": "SVGS"
    },
    "campoLivre": "",
    "txId": "h7a786d8a7s6gd1hgs",
    "dataHoraPagamento": "2022-07-23T19:47:18.108Z"
}
```

### Outros Endpoints

Para facilitar os testes e verificação de conformidade, é importante a implementação de outros endpoints para simular a entrada de novas mensagens Pix.

#### POST `/api/util/msgs/{number}`
Esse endpoint será utilizado para inserir na base de dados, novas mensagens Pix. A quantidade de mensagens a serem adicionadas será de acordo com o pathParam `number`. As informações das mensagens devem ser geradas de forma aleatória, seguindo o padrão dos dados de exemplo.


## Tecnologias

De preferência, se você tiver domínio, esperamos ver o backend em NodeJS ou Django, utilizando PostgreSQL.


### O que nós esperamos ver no seu desafio

* Ver a utilização do framework da melhor forma possível (metodologia/estrutura).
* Ver a utilização de dependency managers (npm, webpack, pip, yarn).
* Projeto organizado de acordo com as responsabilidades de cada componente.
* Rotas de APIs bem estruturadas.
* Testes unitários e/ou testes de integração

### O que nos impressionaria
* Ver o código rodando live (Bucket estático S3, Heroku, Firebase Hosting).

### O que nós não gostaríamos

* Descobrir que não foi você quem fez seu desafio :(
* Ver commits grandes, sem muita explicação nas mensagens em seu repositório 
* Não conseguir rodar a sua aplicação por algum erro de compilação

## O que avaliaremos de seu teste

* Histórico de commits do git
* As instruções de como rodar o projeto
* Estruturação do projeto
* Organização, semântica, estrutura, legibilidade, manutenibilidade do seu código
* Alcance dos objetivos propostos

