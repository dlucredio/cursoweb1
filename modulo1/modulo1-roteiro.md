# Desenvolvimento de Software para Web 1 - Prof. Daniel Lucrédio
## Módulo 1 - Exemplos

### Demonstração 1 - HTTP do lado do cliente e servidor
---

1. Abrir um terminal e digitar:

```zsh
telnet dlucredio.com 80
...
GET /index.html HTTP/1.1
Host:dlucredio.com

(dar dois enters)
...
````

Alternativamente, para websites que possuem SSL:

```sh
openssl s_client -connect pt.wikipedia.org:443
(aguardar a conexão e as credenciais aparecerem)

GET /wiki/Fingolfin HTTP/1.1
Host:pt.wikipedia.org

(dar dois enters)
...
```

2. Criar um novo projeto Java no NetBeans, chamado GatoTom
3. Criar uma classe chamada GatoTom

```java
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.StringTokenizer;

public class GatoTom extends Thread {
    private static String PASTA_WWW = <caminho para uma pasta www>;
    public void run() {
        try {
            ServerSocket serversocket = new ServerSocket(8080);
            System.out.println("Aguardando requisições");
            while (true) {
                Socket connectionsocket = serversocket.accept();
                BufferedReader input = new BufferedReader(new InputStreamReader(connectionsocket.getInputStream()));
                DataOutputStream output = new DataOutputStream(connectionsocket.getOutputStream());
                tratarRequisicao(input, output);
                output.flush();
                output.close();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private void tratarRequisicao(BufferedReader input, DataOutputStream output) throws Exception {
        String tmp = input.readLine();
        if (tmp != null && tmp.startsWith("GET")) {
            StringTokenizer st = new StringTokenizer(tmp);
            st.nextToken();
            String arquivo = st.nextToken().substring(1);
            System.out.println("Requisição: " + arquivo);
            File f = new File(PASTA_WWW + File.separator + arquivo);
            if (!f.exists() || !f.isFile()) {
                String resposta = "HTTP/1.0 404 Not Found\r\n";
                resposta += "Connection: close\r\n";
                resposta += "Server: Exemplo\r\n";
                resposta += "\r\n";
                output.writeBytes(resposta);
                System.out.println("Arquivo não existe");
                return;
            }
            FileInputStream fis = new FileInputStream(f);
            String resposta = "HTTP/1.0 200 OK\r\n";
            resposta += "Connection: close\r\n";
            resposta += "Server: Exemplo\r\n";
            if (arquivo.endsWith(".html")) {
                resposta += "Content-Type: text/html\r\n";
            } else if (arquivo.endsWith(".jpg")) {
                resposta += "Content-Type: image/jpeg\r\n";
            } else if (arquivo.endsWith(".css")) {
                resposta += "Content-Type: text/css\r\n";
            }
            resposta += "\r\n";
            output.writeBytes(resposta);
            while(true) {
                int b = fis.read();
                if(b == -1) break;
                output.write(b);
            }
            fis.close();
        }
    }
    public static void main(String[] args) {
        new GatoTom().start();
    }
}
```

4. Mostrar a pasta www, criar um arquivo teste.html com conteúdo textual qualquer dentro, testar no browser
5. Criar uma imagem .jpg qualquer, salvar na pasta www com o nome “imagem.jpg”, e testar no browser
6. Fim



### Demonstração 2 - Exemplo de geração de conteúdo dinâmico + interativo
---

1. Rodar o servidor da demonstração 1
2. Criar, na pasta www, um arquivo HTML: teste.hml

```html
<html>
    <head>
      <title>Este é o título da página</title>
      <meta charset="UTF-8">
    </head>
    <body>
        <h1>Cadastro</h1>
        <hr>
        <p>Digite seus dados.</p>
        <form action="executar" method="GET">
              E-mail: <input name="email" type="text" /><br/>
              Senha: <input name="senha" type="password" /><br/>
              Sexo: <input type="radio" name="sexo" value="masculino" /> Masculino
              <input type="radio" name="sexo" value="feminino" /> Feminino
              <br/>
              Receber notícias:
              <input type="checkbox" name="cinema" value="cinema" />
              <br />
              Cadastrar meu e-mail no site:
              <input type="checkbox" name="cadastrar" value="cadastrar" />
              <br />
              <input type="submit" value="Enviar" />
        </form>
    </body>
</html>
```
  
3. Abrir no browser, e simular um cadastro. Mostrar a janela do servidor, o que foi enviado
4. Modificar o código do servidor, para tratar a requisição, e testar novamente

```java
private void tratarRequisicao(BufferedReader input, DataOutputStream output) throws Exception {
  String tmp = input.readLine();
  if (tmp.startsWith("GET")) {
     StringTokenizer st = new StringTokenizer(tmp);
     st.nextToken();
     String arquivo = st.nextToken().substring(1);
     System.out.println("Requisição: " + arquivo);
            if (arquivo.startsWith("executar")) {
                String email = "";
                String conteudoForm = arquivo.substring(9);
                StringTokenizer st2 = new StringTokenizer(conteudoForm, "&");
                while (st2.hasMoreTokens()) {
                    String s = st2.nextToken();
                    int i = s.indexOf('=');
                    String nomeParam = s.substring(0, i);
                    String valorParam = s.substring(i + 1);
                    if (nomeParam.equals("email")) {
                        email = valorParam;
                    }
                    System.out.println("INSERT INTO TABELA " + nomeParam + " = " + valorParam);
                }
                String resposta = "HTTP/1.0 200 OK\r\n";
                resposta += "Connection: close\r\n";
                resposta += "Server: Exemplo\r\n";
                resposta += "Content-Type: text/html\r\n";
                resposta += "\r\n";
                resposta += "<html>";
                resposta += "    <head>";
                resposta += "      <title>Este é o título da página</title>";
                resposta += "      <meta charset=\"UTF-8\">";
                resposta += "    </head>";
                resposta += "    <body>";
                resposta += "        <h1>Cadastro</h1>";
                resposta += "        <hr>";
                resposta += "        <p>Cadastro do e-mail " + email + " bem-sucedido!</p>";
                resposta += "    </body>";
                resposta += "</html>";
                output.writeBytes(resposta);
            } else {

     ...
```

5. Mudar para gerar conteúdo interativo

```java
resposta += "    <body>";
resposta += "        <h1>Cadastro</h1>";
resposta += "        <hr>";
resposta += "        <div id=\"resultado\">Cadastro do e-mail " + email + " bem-sucedido!</div>";
resposta += "        <button onclick=\"document.getElementById('resultado').style.display='none';\">Dispensar</button>";
resposta += "    </body>";
```

6. Mudar o método do formulário para POST (no arquivo teste.html) e testar
7. Modificar o servidor GatoTom (da demonstração 1) para tratar requisições POST

```java
private void tratarRequisicao(BufferedReader input, DataOutputStream output) throws Exception {
    String tmp = input.readLine();
    if (tmp != null && tmp.startsWith("POST")) {
        int tamanho = 0;
        while (true) {
            System.out.println(tmp);
            tmp = input.readLine();
            // if(tmp.startsWith("Content-Length:")) {
            //     tamanho = Integer.parseInt(tmp.substring(16));
            // }
            if (tmp.trim().isEmpty()) {
                //    System.out.println("Início conteúdo POST ("+tamanho+" bytes):");
                //    for(int i=0;i<tamanho;i++) {
                //        int b = input.read();
                //        System.out.print((char) b);
                //    }
                //    System.out.println("\nFim conteúdo POST");
                return;
            }
        }
    }
    else if (tmp != null && tmp.startsWith("GET")) {
    ...
```

8. Testar no browser
9. Descomentar as linhas acima, e testar
10. Fim
