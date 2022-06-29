# Simple URL Shortener Using Microservices Architecture

>Python + PostgreSQL + RabbitMQ + Redis + Docker

 1️⃣ A aplicação será composta por dois serviços. O primeiro serviço é um serviço de gerenciamento e o outro serviço é o serviço de redirecionamento. 

 2️⃣ RabbitMQ será usado para transferir mensagens entre serviços

```mermaid

graph TD;
    ManagementService.->RabbitMQ[("🐇 RabbitMQ")];
    RabbitMQ.->RedirectionService
    Clientes("👨‍🚀 Clientes")---ManagementService["🌎 ManagementService"]
    Clientes---RedirectionService["🌎 RedirectionService"];
    ManagementService---Database[("🐘 PostgreSQL")];    
    RedirectionService---Redis[("🧊 Redis")]
    RedirectionService---Database[("🐘 PostgreSQL")]
    
```

## ManagementService (Serviço de Gestão)

O Management Service possui uma API RESTful para criar e excluir URLs. 
* O serviço de gerenciamento deve usar PostgreSQL para a camada de persistência. 
* Após criar ou deletar uma URL curta, as informações devem ser enviadas para o RabbitMQ

### **Rotas:**
#### **Rota de criação**

* A rota deve criar uma URL curta baseado em uma URL real. 
* A URL curta deve ser exclusiva.
* A identificação de hash da URL curta deve estar localizada no caminho da URI. **Exemplo:** http://localhost:8080/uAYC3sOddP

_Exemplo de solicitação_
~~~json
{ 
  "realURL": "https://www.example.com/test" 
}
~~~


_Exemplo de resposta_
~~~json
{ 
  "id": 3, 
  "realURL": "https://www.example.com/test", 
  "shortURL": "http://localhost:8080/gfjhgESta" 
}
~~~

#### **Rota de exclusão:**

* Remove a URL curta usando o id.

_Exemplo de solicitação_
~~~
http://localhost:8080/3
~~~

_Exemplo de resposta_
~~~json
{ 
  "Success"
}
~~~

## RedirectionService (Serviço de redirecionamento)
O serviço encontrará a URL real, com base na parte de hash da URL curta e o usuário será redirecionado para a URL real. O redirecionamento aceita informações sobre URLs curtas por meio do RabbitMQ.

No caso de criação de uma URL curta no Serviço de Gerenciamento, as informações serão armazenadas no Redis no Serviço de Redirecionamento, enquanto no caso de exclusão de uma URL curta, as informações serão excluídas do Serviço de Redirecionamento no Redis. O serviço de redirecionamento tem uma rota de API RESTful.

### **Rota de redirecionamento**

* A rota deve retornar um código HTTP 302 para o URL curto existente.
* A rota deve retornar um código HTTP 404 para URLs curtos inexistentes.
* Limitador de solicitações
  * Implemente a limitação de taxa no serviço de redirecionamento onde o serviço permite 10 solicitações de redirecionamento em um período de 120 segundos para uma URL específica.
  * A rota deve retornar o código HTTP 429 após atingir o limite.
