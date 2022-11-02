# NGINX Rootless

É inegável que a preocupação com a segurança deve estar em todas as etapas da
entrega da aplicação.

Os arquivos desse repositório, por exemplo, exemplificam como podemos "subir"
um serviço do nginx, utilizando containers, sem utilizar o usuário root.

Apesar de mais controlado, o ambiente com containers não é inviolável e por
isso também precisamos tomar os mesmos cuidados que em um ambiente clássico de
servidores, sempre manter atualizado com o minimo de ferramentas e permissões
necessárias para a aplicação.

> **Ok, mas se vamos precisar ter as mesmas preocupações por que escolher
utilizar um ambiente com containers, ao invés dos clássicos servidores?**

Essa é fácil. A possibilidade de escalabilidade e facilidade para manter os
componentes sempre atualizados, além da velocidade de entrega muito maior,
devido aos mínimos pontos apresentados. E isso é só a ponta do *iceberg*.

> **Certo, já que é tão melhor vou usar containers. Qual o passo inicial para ter
um ambiente seguro?**

Um dos pontos mais simples de serem implementados, e que independem da
aplicação, na maioria dos casos, são as permissões.

> **Que tipo de permissão?**

Voltando aos cuidados que teríamos, caso fosse um ambiente de servidores
clássicos. Normalmente, um dos passos ao configurar um serviço é criar um
usuário no sistema que será responsável por executar a aplicação,
principalmente em serviços *web*, como é o caso do *nginx*. Isso acontece como
formar de dificultar uma possível exploração de vulnerabilidade. Ataques que
têm como vetor inicial os serviços *web*, são clássicos e sempre abordados
nos testes de intrusão.

> **Uau, já que é assim tão conhecido, o que posso fazer para prevenir isso?**

Como ia dizendo, criar um usuário para executar o tal serviço. No nosso, caso o
nginx.

> **Mas pelo que sei todos os containers utilizam o usuário root**

De fato, a grande maioria dos containers são disponibilizados com o usuário
root. Mas no nginx, por exemplo, precisamos fazer uma alteração mínima para
conseguir rodar sem o root.

> **Ié? Que alteração é essa?**

Apenas mudar o local onde o arquivo de pid para processo que executará o
serviço está, e o contexto dos arquivos temporários utilizados pelo *nginx*.


```conf
...
# Change pid file location: new location at `conf.d/change-pid-location.conf`
pid         /tmp/nginx.pid;

http {
    # Add new temp locations
    client_body_temp_path /tmp/client_temp;
    proxy_temp_path       /tmp/proxy_temp_path;
    fastcgi_temp_path     /tmp/fastcgi_temp;
    uwsgi_temp_path       /tmp/uwsgi_temp;
    scgi_temp_path        /tmp/scgi_temp;
...
```

E apontar no Dockerfile, qual usuário que deverá ser utilizado no container.

```dockerfile
USER nginx
```

Com essas minimas modificações você estará rodando o *nginx* com um usuário não
root.

## Build

```
docker docker build -t nginx-rootless:0.1.0 .
```

## Run

```
docker run --rm --name nginx-rootless -p 80:80 -it nginx-rootless:0.1.0
```
