# Linha de comando
### Instalar o Mountebank em uma máquina local:
```
npm i mountebank
```

### Subir o servidor do Mountebank:
```
mb
```

### Salvar configuração de sessão para inicializar em uma nova sessão posteriormente:
(ex: gravação de proxies e stubs criados via chamadas de API REST p/ o Mountebank)
```
mb save --port 2525 --savefile <name_of_the_file>.json --removeProxies
```

### Inicializar o servidor Mountebank com um arquivo JSON específico:
```
mb restart --port 2525 --configfile <name_of_the_file>.json
```

### Trocar de modo gravação para modo replay (após uso de proxies):
mb replay --port 2525s

# Proxy
Proxies devem ser criados utilizando a API REST do Mountebank, para isto o servidor deverá estar de pé.
Como exemplo, para criar um Proxy para copiar uma vez a primeira chamada realizada para uma API, o seguinte deve ser feito:
```
curl --location --request POST 'http://127.0.0.1:2525/imposters' \
--header 'Content-Type: application/json' \
--data-raw '{
   "port":10000,
   "protocol":"http",
   "name":"proxyOnce",
   "recordRequests": true,
   "stubs":[
      {
         "responses":[
            {
               "proxy":{
                  "to":"https://serverest.dev",
                  "mode":"proxyOnce",
                  "predicateGenerators":[
                     {
                        "matches":{
                           "method":true,
                           "path":true,
                           "body":true
                        },
                        "caseSensitive":true
                     }
                  ]
               }
            }
         ]
      }
   ]
}'
```

Após criar o Proxy, a ou as chamadas que seriam feitas para a aplicação original dependendo do modelo do Proxy (proxyOnce ou proxyMany), devem ser feitas para o servidor Mountebank fazendo uso da porta especificada na chamada anterior:
```
curl --location --request POST 'http://127.0.0.1:10000/login' \
--header 'Content-Type: application/json' \
--data-raw '{
    "email": "fulano@qa.com",
    "password": "teste"
}'
```

Isto irá garantir que o Mountebank fará a chamada para a API ou endereço-alvo, e irá gravar todo processo de request/response a ser replicado e virtualizado.
