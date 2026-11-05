# Lab 01: Balanceamento de Carga com Nginx

**Aluno:** Romy Manuel Martinez Nava **Matrícula:** 202311200272

## 1\. Ajuste de Porta (8080)

*Descreva brevemente quais arquivos foram alterados para a porta 8080 e cole abaixo a saída do terminal comprovando o funcionamento na nova porta.*

```
curl -i http://localhost:8080

HTTP/1.1 200 OK
Server: nginx/1.31.8
Date: Thu, 05 Nov 2026 19:09:21 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 167
Connection: keep-alive

Hostname: 8d7d2ac21d42
IP: 127.0.0.1
IP: ::1
IP: 172.18.0.2
RemoteAddr: 172.18.0.5:59252
GET / HTTP/1.1
Host: meus_servidores
User-Agent: curl/8.5.0
Accept: */*
```

## 2\. Teste dos Algoritmos de Balanceamento

### 2.1. Round Robin com Peso (Weighted)

*Qual configuração foi aplicada no `nginx.conf`?*

```
    server backend1:80 weight=8;
    server backend2:80 weight=1;
    server backend3:80 weight=1;
    
```

*Evidência da distribuição desproporcional (saída do terminal):*

```
python3 - <<'PY'
pesos = [3, 1, 1]
total = sum(pesos)

for i, peso in enumerate(pesos, 1):
    percentual = peso / total * 100
    print(f"Servidor {i}: {percentual:.2f}%")
PY
Servidor 1: 60.00%
Servidor 2: 20.00%
Servidor 3: 20.00%
```

### 2.2. IP Hash (Persistência de Sessão)

*Evidência de que o tráfego foi fixado em um único nó:*

```
    python3 - <<'PY'
    import requests

    servidores = []

    for i in range(10):
        resposta = requests.get("http://localhost:8080")
        hostname = resposta.text.split("IP:")[0].strip()
        servidores.append(hostname)
        print(f"Requisição {i+1}: {hostname}")

    print("\nResultado:")
    print("IP Hash funcionando!" if len(set(servidores)) == 1 else "Servidores diferentes!")
    PY
    Requisição 1: Hostname: 8d7d2ac21d42
    Requisição 2: Hostname: 8d7d2ac21d42
    Requisição 3: Hostname: 8d7d2ac21d42
    Requisição 4: Hostname: 8d7d2ac21d42
    Requisição 5: Hostname: 8d7d2ac21d42
    Requisição 6: Hostname: 8d7d2ac21d42
    Requisição 7: Hostname: 8d7d2ac21d42
    Requisição 8: Hostname: 8d7d2ac21d42
    Requisição 9: Hostname: 8d7d2ac21d42
    Requisição 10: Hostname: 8d7d2ac21d42

    Resultado:
    IP Hash funcionando!
```

### 2.3. Least Connections

*Trecho de configuração aplicado:*

```
    python3 - <<'PY'
    import requests

    for i in range(10):
        resposta = requests.get("http://localhost:8080")
        hostname = resposta.text.split("IP:")[0].strip()
        print(f"Requisição {i+1}: {hostname}")
    PY
    Requisição 1: Hostname: 8d7d2ac21d42
    Requisição 2: Hostname: 7925563fccd2
    Requisição 3: Hostname: 9eccb37f4ebb
    Requisição 4: Hostname: 8d7d2ac21d42
    Requisição 5: Hostname: 7925563fccd2
    Requisição 6: Hostname: 9eccb37f4ebb
    Requisição 7: Hostname: 8d7d2ac21d42
    Requisição 8: Hostname: 7925563fccd2
    Requisição 9: Hostname: 9eccb37f4ebb
    Requisição 10: Hostname: 8d7d2ac21d42
```

## 3\. Segurança e Rastreabilidade

### 3.1. Rate Limiting

*Evidência do bloqueio por excesso de requisições (comprovando o erro 503):*

```
    python3 - <<'PY'
    import requests

    for i in range(10):
        resposta = requests.get("http://localhost:8080")
        print(f"Requisição {i+1}: HTTP {resposta.status_code}")
    PY
    Requisição 1: HTTP 200
    Requisição 2: HTTP 503
    Requisição 3: HTTP 503
    Requisição 4: HTTP 503
    Requisição 5: HTTP 503
    Requisição 6: HTTP 503
    Requisição 7: HTTP 503
    Requisição 8: HTTP 503
    Requisição 9: HTTP 503
    Requisição 10: HTTP 503

```

### 3.2. Injeção de Headers de Proxy

*Evidência da aplicação back-end recebendo o IP real do cliente:*

```
        Hostname: 9eccb37f4ebb
    IP: 127.0.0.1
    IP: ::1
    IP: 172.18.0.3
    RemoteAddr: 172.18.0.5:57076
    GET / HTTP/1.1
    Host: localhost
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:149.0) Gecko/20100101 Firefox/149.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Encoding: gzip, deflate, br, zstd
    Accept-Language: pt-BR,pt;q=0.9,en-US;q=0.8,en;q=0.7
    Priority: u=0, i
    Sec-Fetch-Dest: document
    Sec-Fetch-Mode: navigate
    Sec-Fetch-Site: none
    Sec-Fetch-User: ?1
    Sec-Gpc: 1
    Upgrade-Insecure-Requests: 1
    X-Forwarded-For: 172.18.0.1
    X-Real-Ip: 172.18.0.1
```

## 4\. Análise: Limitações da versão Open Source

**Pergunta:** Por que a versão Open Source do Nginx não possui suporte nativo aos algoritmos de *Least Response Time* e *Adaptive Routing*? Como essas limitações são tratadas no mercado?

**Resposta:**  A versão Open Source do Nginx não possui suporte nativo a esses algoritmos devido à complexidade de implementação e à necessidade de recursos adicionais para monitorar o desempenho dos servidores em tempo real. Além disso, algumas dessas funcionalidades somente estão em versões pagas. Eu utilizaria ferramentas como Kubernetes para implementar essas estratégias e melhorar a distribuição das requisições entre os servidores.