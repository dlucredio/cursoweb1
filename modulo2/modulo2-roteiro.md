# Desenvolvimento de Software para Web 1 - Prof. Daniel Lucrédio
## Módulo 2 - Exemplos

### Demonstração 1 - Instalando o Tomcat
---

1. Mostrar o arquivo baixado
2. Descompactar em uma pasta sem espaços ou acentos
3. Mostrar a estrutura de diretórios
4. Mostrar server.xml
5. Mostrar tomcat-users.xml
6. Rodar o tomcat  
7. No windows
- Abrir o Windows PowerShell  
- Rodar:    
```
$env:JAVA_HOME="C:\<caminho_para_java>\jreXXX"
```

- Executar startup.bat
8. No Linux/Mac
- Abrir um terminal
- Rodar:

```sh  
echo $JAVA_HOME
```
ou

```sh
export JAVA_HOME="/<caminho_para_java>/jreXXX"
```

- Executar startup.sh (pode ser necessário dar permissão)

9. Abrir localhost:8080
- Mostrar que não tem acesso no manager
- Modificar tomcat-users.xml, adicionando a seguinte linha (obs: será importante na execução dos laboratórios)

```xml
<user username="admin" password="admin" roles="manager-gui" />
```
- Parar tomcat e reiniciar, e tentar novamente

10. Mostrar aplicações “manager” e “status”
11. Fim

### Demonstração 2 - Aplicação “Alo mundo” em linha de comando
---

1. Criar uma pasta AplicacaoAloMundo
2. Criar uma pasta src/br/ufscar/dc/web1/alomundo
3. Criar arquivo texto AloMundoServlet.java

```java
package br.ufscar.dc.web1.alomundo;

import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class AloMundoServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request,
              HttpServletResponse response)
                 throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        out.println("<html>");
        out.println("<head>");
        out.println("<title>Primeira aplicação</title>");
        out.println("</head>");
        out.println("<body>");
        out.println("<h1>Alo mundo!</h1>");
        out.println("</body>");
        out.println("</html>");
        out.close();
    }
}
````

4. Compilar

```sh
javac -cp /<caminho_para_tomcat>/lib/servlet-api.jar AloMundoServlet.java
```

5. Mostrar que gerou um .class
6. Montar a estrutura da aplicação
- Criar a pasta WEB-INF/classes/br/ufscar/dc/web1/alomundo
- Mover o .class para WEB-INF/classes/br/ufscar/dc/web1/alomundo
- Criar o arquivo web.xml na pasta WEB-INF
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app>
    <servlet>
        <servlet-name>AloMundo</servlet-name>
        <servlet-class>br.ufscar.dc.web1.alomundo.AloMundoServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>AloMundo</servlet-name>
        <url-pattern>/TestarAloMundo</url-pattern>
    </servlet-mapping>
    <session-config>
        <session-timeout>
            30
        </session-timeout>
    </session-config>
</web-app>
```

7. Criar o arquivo .war (executar a partir do Desktop, ou pasta que contem AplicacaoAloMundo)

```sh
<caminho_para_java>/jar -cvf AplicacaoAloMundo.war -C ./AplicacaoAloMundo .
```

8. Fazer o deploy
- Rodar o tomcat
- Entrar no localhost:8080  e fazer o deploy pela aplicação “manager”
9. Testar a aplicação, abrindo a URL ```http://localhost:8080/AplicacaoAloMundo/TestarAloMundo```
- Fazer o undeploy da aplicação, pela aplicação "manager"
10. Apagar o arquivo web.xml
11. Modificar o código fonte, adicionando as seguintes linhas:

```diff
package br.ufscar.dc.web1.alomundo;

import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
+import javax.servlet.annotation.WebServlet;

+@WebServlet(urlPatterns = {"/AloMundoServlet"})
public class AloMundoServlet extends HttpServlet {
...
```

12. Refazer o processo e testar, abrindo agora a URL ```http://localhost:8080/AplicacaoAloMundo/AloMundoServlet```
13. Fazer undeploy da aplicação e parar o tomcat
14. Fim

### Demonstração 3 - Aplicação “Alo mundo” com Apache Maven
---

1. Abrir um terminal e executar o seguinte comando (não esquecer de configurar o Maven):

```sh
mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-webapp -DarchetypeVersion=1.4
```

- groupId: ```br.ufscar.dc.web1```
- artifactId: ```alo-mundo-web-maven```
- Não modificar as outras opções. Será criado um diretório chamado alo-mundo-web-maven. Abri-lo em um editor de texto (ex: Visual Studio Code)

2. Vamos criar um Maven Wrapper para que o projeto tenha sua própria instalação do Maven, fixando assim a versão e evitando problemas de versionamento
- Abrir um terminal dentro da pasta do projeto (alo-mundo-web-maven) e executar o seguinte comando

```sh
mvn -N io.takari:maven:wrapper
```

3. Adicionar a seguinte dependência e plugin ao arquivo pom.xml

```diff
<project ... >
   <dependencies>
      ...
+      <dependency>
+         <groupId>javax.servlet</groupId>
+         <artifactId>javax.servlet-api</artifactId>
+         <version>4.0.0</version>
+         <scope>provided</scope>
+      </dependency>
   </dependencies>
   ... 
   <build>
      ...
+      <plugins>
+         <plugin>
+            <groupId>org.apache.tomcat.maven</groupId>
+            <artifactId>tomcat7-maven-plugin</artifactId>
+            <version>2.2</version>
+            <configuration>
+               <url>http://localhost:8080/manager/text</url>
+                  <server>Tomcat9</server>
+                  <path>/${project.artifactId}</path>
+            </configuration>
+         </plugin>
+      </plugins>
-      <pluginManagement> (atenção para o local correto, NÃO é aqui dentro!)
         <plugins>
            ...
         </plugins>
      </pluginManagement>

      ...
   </build>
   ...   
</project>
````

- Agora é preciso inserir as credenciais do Tomcat para permitir que o plugin do Maven acesse o servidor
- Editar o arquivo tomcat-users.xml, dentro da instalação do Tomcat, e adicionar o suporte para o papel de script para o usuário criado anteriormente:

```xml
<user username="admin" password="admin" roles="manager-gui,manager-script" />
```

- Agora vamos configurar o Maven para gravar usuário e senha do Tomcat. Essa configuração deve ficar FORA do projeto, pois é algo específico de uma determinada máquina, e portanto caso esteja sendo utilizado repositório para compartilhamento de código, NÃO deve ser compartilhado!
- Abrir (ou criar) o arquivo .m2/settings.xml que fica localizado dentro da pasta de usuário. Ex: /Users/daniellucredio/.m2/settings.xml
- O conteúdo importante é o que está destacado abaixo. Insira no local correto (dentro de servers). Caso não exista, crie o conteúdo todo e as tags pais, caso necessário:

```diff
<?xml version="1.0" encoding="UTF-8"?>
<settings>
    <servers>
+        <server>
+            <id>Tomcat9</id>
+            <username>admin</username>
+            <password>admin</password>
+        </server>
    </servers>
</settings>
```

4. Criar uma nova pasta dentro de src/main: java/br/ufscar/dc/web1/alomundo
5. Criar um arquivo dentro da pasta acima, chamado AloMundoServlet.java:

```java
package br.ufscar.dc.web1.alomundo;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet(urlPatterns = { "/AloMundoServletNoMaven" })
public class AloMundoServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(final HttpServletRequest req, final HttpServletResponse resp)
            throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");
        PrintWriter out = resp.getWriter();
        out.println("<html>");
        out.println("<head>");
        out.println("<title>Primeira aplicação com Maven</title>");
        out.println("</head>");
        out.println("<body>");
        out.println("<h1>Alo mundo com Maven!</h1>");
        out.println("</body>");
        out.println("</html>");
        out.close();

    }
}
```

6. Abrir um terminal e executar o Tomcat. Opcionalmente, pode-se executar um comando de monitoramento do arquivo de log, para visualizar mensagens em tempo real:

```sh
/home/daniel/Programs/apache-tomcat-9.0.31/bin/startup.sh
tail -f /home/daniel/Programs/apache-tomcat-9.0.31/logs/catalina.out
````

7. Abrir um terminal dentro da pasta do projeto e executar os seguintes comandos (note que devido ao uso do Maven Wrapper, não é mais necessário acessar a instalação global do Maven):

```
./mvnw tomcat7:deploy
./mvnw tomcat7:redeploy <- este comando automaticamente faz o deploy também, caso não exista
./mvnw tomcat7:undeploy
````

8. Testar, abrindo o browser no endereço ```http://localhost:8080/alo-mundo-web-maven/AloMundoServletNoMaven```

9. Caso queira fazer o deploy manualmente, a pasta “target” dentro do projeto irá conter o arquivo .war gerado, após a execução de um dos comandos acima
10. Caso queira apenas gerar o .war (sem fazer deploy), rodar o comando:

```sh
./mvnw package
```

### Demonstração 4 - Aplicação “Alo mundo” com NetBeans
---

1. Abrir o NetBeans
2. Criar um novo projeto Java with Maven -> Web Application
- Project name: ```alomundo-web-nb```
- Group id: ```br.ufscar.dc.web1```
- Não adicionar nenhum servidor
3. No arquivo pom.xml, adicionar o plugin do Tomcat para o Maven
- Veja que aqui não é necessário adicionar as dependências para a servlet API, como no exemplo anterior

```diff
<project ... >
   ... 
   <build>
      ...
      <plugins>
+         <plugin>
+            <groupId>org.apache.tomcat.maven</groupId>
+            <artifactId>tomcat7-maven-plugin</artifactId>
+            <version>2.2</version>
+            <configuration>
+               <url>http://localhost:8080/manager/text</url>
+                  <server>Tomcat9</server>
+                  <path>/${project.artifactId}</path>
+            </configuration>
+         </plugin>
      </plugins>
      ...
   </build>
   ...   
</project>
```

4. Caso já tenha modificado o arquivo settings.xml do Maven no exemplo anterior, não é necessário fazê-lo aqui. Mas caso precise, o arquivo se encontra visível a partir da raiz do projeto no NetBeans, em “Project Files” (ele aponta para o arquivo na pasta do usuário)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings>
    <servers>
        <server>
            <id>Tomcat9</id>
            <username>admin</username>
            <password>admin</password>
        </server>
    </servers>
</settings>
```

5. Adicionar um novo Servlet: ```br.ufscar.dc.web1.alomundonb.AloMundoServlet```
6. No novo servlet, adicionar um texto de exemplo para ser impresso
7. Para fazer o deploy, seguir os seguintes passos:
- Iniciar o Tomcat em um terminal
- Clicar com o botão direito no projeto, no NetBeans e executar “Run Maven” -> “Goals”
- Em Goals, especificar: ```tomcat7:redeploy```
- Marcar a opção “Remember As” e digitar um texto (ex: tomcat7:redeploy mesmo) para facilitar a repetição
- Alternativamente, é possível executar o Goal selecionando o projeto, e acessando a janela “Navigator” (abaixo de Projects)
8. Testar a URL ```http://localhost:8080/alomundo-web-nb/AloMundoServlet```
9. Modificar a classe do servlet, gerando um texto diferente e um comentário HTML
10. Testar novamente (basta executar o mesmo Maven Goal novamente)
11. Para remover, executar o Maven Goal ```tomcat7:undeploy```
12. Caso queira apenas obter o arquivo .war para fazer deploy, o mesmo se encontra na pasta “target” do projeto a cada compilação


### Demonstração 5 - Aplicação “Alo mundo” com IntelliJ
---

1. Abrir o IntelliJ
2. Criar um novo projeto Java -> Maven -> Create from Archetype -> org.apache.maven.archetypes:maven-archetype-webapp
- Project name: ```alomundo-web-ij```
- Group id: ```br.ufscar.dc.web1```
3. Ao abrir o projeto, será apresentada uma opção para importar as mudanças feitas nos arquivos de projeto. Escolher: “Enable Auto-Import”
4. No arquivo pom.xml, adicionar a dependência para a Servlet-API e o plugin do Tomcat para o Maven

```diff
<project ... >
   <dependencies>
      ...
+      <dependency>
+         <groupId>javax.servlet</groupId>
+         <artifactId>javax.servlet-api</artifactId>
+         <version>4.0.0</version>
+         <scope>provided</scope>
+      </dependency>
   </dependencies>
   ... 
   <build>
      ...
+      <plugins>
+         <plugin>
+            <groupId>org.apache.tomcat.maven</groupId>
+            <artifactId>tomcat7-maven-plugin</artifactId>
+            <version>2.2</version>
+            <configuration>
+               <url>http://localhost:8080/manager/text</url>
+                  <server>Tomcat9</server>
+                  <path>/${project.artifactId}</path>
+            </configuration>
+         </plugin>
+      </plugins>
-      <pluginManagement> (atenção para o local correto, NÃO é aqui dentro!)
         <plugins>
            ...
         </plugins>
      </pluginManagement>

      ...
   </build>
   ...   
</project>
```

5. Caso já tenha modificado o arquivo settings.xml do Maven no exemplo anterior, não é necessário fazê-lo aqui. Mas caso precise, o arquivo se encontra visível a partir da raiz do projeto, clicando com o botão direito no projeto, e escolhendo “Maven” ->  “Open settings.xml” (ele aponta para o arquivo na pasta do usuário)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings>
    <servers>
        <server>
            <id>Tomcat9</id>
            <username>admin</username>
            <password>admin</password>
        </server>
    </servers>
</settings>
```

6. Criar uma nova pasta “java” dentro de “src/main” (o archetype do Maven não faz isso)
7. Abrir o menu “File” -> “Project Structure” -> “Modules” -> “Sources”
8. Escolher a pasta “java” criada e clicar em “Mark as: Sources” (a pasta ficará registrada como uma pasta de código-fonte, e mudará de cor, além de aparecer no lado direito como código-fonte)
9. Voltar para a tela principal, clicar com o botão direito sobre a pasta “java” e criar nova classe: ```br.ufscar.dc.web1.alomundo.AloMundoServlet```

```java
package br.ufscar.dc.web1.alomundo;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet(urlPatterns = { "/AloMundoServletNoIntelliJ" })
public class AloMundoServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(final HttpServletRequest req, final HttpServletResponse resp)
            throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");
        PrintWriter out = resp.getWriter();
        out.println("<html>");
        out.println("<head>");
        out.println("<title>Primeira aplicação com IntelliJ</title>");
        out.println("</head>");
        out.println("<body>");
        out.println("<h1>Alo mundo com IntelliJ e Maven!</h1>");
        out.println("</body>");
        out.println("</html>");
        out.close();
    }
}
```

- Para facilitar a edição de código Java, é possível habilitar os autoimports: menu File -> Settings -> Editor -> General -> Auto Import -> Java -> Add unambiguous imports on the fly e Optimize imports on the fly (for current project)
10. Para fazer o deploy, seguir os seguintes passos:
- Iniciar o Tomcat em um terminal
- Abrir, no lado direito da janela do IntelliJ, a janela “Maven” (caso não apareça, acessar o menu View -> Tool Windows -> Maven)
- Para implantar no Tomcat, escolher o goal Plugins -> tomcat7 -> tomcat7:redeploy
11. Testar a URL ```http://localhost:8080/alomundo-web-ij/AloMundoServletNoIntelliJ```
12. Modificar a classe do servlet, gerando um texto diferente e um comentário HTML
13. Testar novamente (basta executar o mesmo Maven Goal novamente)
14. Para remover, executar o Maven Goal tomcat7:undeploy
15. Caso queira apenas obter o arquivo .war para fazer deploy, escolher o goal Lifecycle -> package. O arquivo .war estará localizado dentro da pasta “target” do projeto


### Demonstração 6 - Obtendo informações de um request
---

1. Criar um projeto Java Web (em qualquer uma das plataformas) chamado
- Group Id: ```br.ufscar.dc.web1```
- Artifact Id: ```demorequest```
2. Adicionar um novo Servlet: ```br.ufscar.dc.web1.demorequest.InterpretarRequestServlet```
3. Acrescentar o seguinte código no corpo do método de tratamento de requisição:

```java
package br.ufscar.dc.web1.demorequest;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet(urlPatterns = { "/InterpretarRequestServlet" })
public class InterpretarRequestServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");
        try (PrintWriter out = resp.getWriter()) {
            String requestURL = req.getRequestURL().toString();
            String protocol = req.getProtocol();
            int port = req.getLocalPort();
            String queryString = req.getQueryString();
            out.println("Requisição: " + requestURL + "<br/>");
            out.println("Protocolo: " + protocol + "<br/>");
            out.println("Porta: " + port + "<br/>");
            out.println("Query: " + queryString + "<br/>");
        }
    }
}
```

4. Mostrar rodando, e adicionar diferentes queries
5. Mostrar que é possível implementar os dois métodos, GET e POST (basta trocar doGet por doPost)
- Opcionalmente, pode-se criar um novo método, chamado processRequest, e redirecionar as chamadas de doGet e doPost para ele
6. Para testar um POST, adicionar uma página HTML, chamada testaRequisicoes.html
7. Adicionar um link para o servlet e testar

```html
<a href="InterpretarRequestServlet?a=123&b=123">Testa</a><br/>
```

8. Adicionar um form para enviar dados via GET e testar

```html
<form name="teste" action="InterpretarRequestServlet">
    Nome: <input type="text" name="nome" /> <br/>
    E-mail: <input type="text" name="email" /> <br/>
    Confirmação de e-mail: <input type="text" name="email" /> <br/>
    Senha: <input type="password" name="senha" /><br/>
    Gênero: <input type="radio" name="genero" value="Masculino" /> Masculino
    <input type="radio" name="genero" value="Feminino" /> Feminino <br/>
    Receber notícias: <input type="checkbox" name="receber" value="claro" /><br/>
    <input type="submit" name="enviar" value="enviado" />
</form>
```

9. Modificar o método para POST e testar. Mostrar que agora a Query String fica null
10. Modificar o código do servlet para mostrar os parâmetros

```java
Map<String,String[]> mapaDeParametros = req.getParameterMap();
Set<Entry<String,String[]>> conjuntoDeParametros = mapaDeParametros.entrySet();
for(Entry<String,String[]> parametro: conjuntoDeParametros) {
    out.println(parametro.getKey()+":");
    for(String v:parametro.getValue()) {
        out.println("["+v+"] ");
    }
    out.println("<br/>");
}
```

11. Mostrar como pegar parâmetros pelo nome

```java
out.println("O nome é: "+req.getParameter("nome")+"<br/>");
boolean receberNoticias = false;
String strReceberNoticias = req.getParameter("receber");
if(strReceberNoticias != null && strReceberNoticias.equals("claro"))
    receberNoticias = true;
out.println("Receber noticias:"+receberNoticias+"<br/>");
```

12. Adicionar um input type hidden e testar

```html
<input type="hidden" name="teste" value="testando" />
```

13. Adicionar um input type file, para mostrar como processar um POST
- Modificar o enctype do form

```html
enctype="multipart/form-data"
```

- Adicionar um novo campo no formulário

```html
Foto: <input type="file" name="arquivoFoto"/> <br/>
```

14. Modificar o servlet para receber o arquivo e gravar no Desktop

```java
File upload = new File("/home/daniel/Desktop/arquivo.txt");
upload.createNewFile();
try (PrintWriter pw = new PrintWriter(new FileWriter(upload))) {
    try (BufferedReader br = req.getReader()) {
        String linha = null;
        pw.println("Conteúdo recebido");
        while ((linha = br.readLine()) != null) {
            pw.println(linha);
        }
        pw.println("Fim do conteúdo");
        pw.flush();
    }
}
```

15. Criar um arquivo texto no desktop só pra ilustrar
16. Testar
- Mostrar que agora, com o novo enctype, os “getParameter” não funcionam mais
- Mostrar que agora tudo deve ser lido e interpretado “na mão”
- Falar que existem frameworks, como o Jakarta Commons, que fazem essa tarefa
17. Mudar o método de novo para GET, e mostrar que dessa vez não dá para ler
18. Fim

### Demonstração 7 - Redirecionamento e encaminhamento
---

1. Abrir uma IDE
2. Criar um novo projeto Web:
- groupId: ```br.ufscar.dc.web1```
- artifactId: ```demonavegacao```
3. Criar um arquivo pagina.html qualquer
4. Adicionar um novo servlet Redirecionador, com o seguinte código:

```java
response.sendRedirect("pagina.html");
```

5. Testar e observar que mudou a URL no browser
6. Adicionar um novo servlet Encaminhador, com o seguinte código:

```java
request.getRequestDispatcher("pagina.html").forward(request, response);
```

7. Testar e observar que não mudou a URL no browser
8. Adicionar um novo servlet ImprimirParametrosServlet, que imprime todos os parâmetros

```java
Map<String,String[]> mapaDeParametros = request.getParameterMap();
Set<Entry<String,String[]>> conjuntoDeParametros = mapaDeParametros.entrySet();
for(Entry<String,String[]> parametro: conjuntoDeParametros) {
    out.println(parametro.getKey()+":");
    for(String v:parametro.getValue()) {
        out.println("["+v+"] ");
    }
    out.println("<br/>");
}
```

9. Modificar os servlets Redirecionador e Encaminhador para chamar o novo servlet ImprimirParametrosServlet
10. Criar dois arquivos html, um ```cabecalho.html``` e um ```rodape.html``` (não usar caracteres especiais, use ```&aacute;``` etc)
11. Criar um novo servlet, chamado TestaIncludeServlet, que faz a inclusão dos dois HTMLs gerados no passo anterior

```java
...
request.getRequestDispatcher("cabecalho.html").include(request, response);
...
request.getRequestDispatcher("rodape.html").include(request, response);
...
```

12. Testar a aplicação
13. Fim


### Demonstração 8 - Atributos no escopo de requisição
---

1. Abrir uma IDE
2. Criar um novo projeto Web:
- groupId: ```br.ufscar.dc.web1```
- artifactId: ```demoescopos```
3. Criar três servlets: ```ServletA```, ```ServletB``` e ```ServletC```
4. O ServletA adiciona um atributo na requisição e faz forward para o ServletB

```java
request.setAttribute("valor", new Integer(100));
request.getRequestDispatcher("ServletB").forward(request, response);
```

5. O ServletB recupera o atributo, modifica-o e o imprime, além de incluir o ServletC

```java
response.setContentType("text/html;charset=UTF-8");
PrintWriter out = response.getWriter();
try {
    out.println("<html>");
    out.println("<head>");
    out.println("<title>Servlet ServletB</title>");
    out.println("</head>");
    out.println("<body>");
    Integer i = (Integer) request.getAttribute("valor");
    i = i + 30;
    request.setAttribute("valor", i);
    out.println("Conteúdo gerado no ServletB: Valor=" + i + "<br/>");
    request.getRequestDispatcher("ServletC").include(request, response);
    out.println("</body>");
    out.println("</html>");
} finally {
    out.close();
}
```

6. O Servlet C recupera o atributo, modifica-o e o imprime

```java
PrintWriter out = response.getWriter();
Integer i = (Integer) request.getAttribute("valor");
i = i + 1000;
request.setAttribute("valor", i);
out.println("Conteúdo gerado no ServletC: Valor=" + i + "<br/>");
```

7. Testar a aplicação e mostrar que o valor está sendo passado de um servlet para outro. Se chamar o ServletB direto irá dar uma exceção.
8. Fim


### Demonstração 9 - Atributos no escopo de aplicação
---

1. Abrir o projeto da demonstração anterior e modificar os códigos dos Servlets
2. O ServletA adiciona um atributo no ServletContext, e mostra na página, caso esteja definido

```java
    getServletContext().setAttribute("valor", new Integer(100));
    ...
    out.println("Página gerada pelo ServletA: ");
    if (getServletContext().getAttribute("valor") == null) {
        out.println("Valor não encontrado!");
    } else {
        Integer i = (Integer) getServletContext().getAttribute("valor");
        out.println("Valor=" + i);
    }
```

3. Os Servlets B e C fazem apenas a exibição na página, de forma similar ao passo 4
4. Testar a aplicação
- Abrir ServletB e/ou ServletC, e notar que não existe valor
- Agora abrir o ServletA antes
- Abrir ServletC em outro browser, para simular outro cliente
5. Fim

### Demonstração 10 - Atributos no escopo de sessão
---

1. Abrir o projeto da demonstração anterior
2. Criar um servlet chamado ```ArmazenarNaSessao```, que pega um parametro chamado nome e armazena na sessão. Em seguida, imprime na página o nome armazenado

```java
String nome = request.getParameter("nome");
request.getSession().setAttribute("nome", nome);
...
out.println("Nome armazenado na sessão:"+nome);
```

3. Criar um servlet ```ExibirSessao``` que exibe os dados da sessão

```java
out.println("Dados da sessão:<br/>");
HttpSession sessao = request.getSession();
String idSessao = sessao.getId();
Date dataCriacao = new Date(sessao.getCreationTime());
Date dataUltimoAcesso = new Date(sessao.getLastAccessedTime());
SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy hh:mm:ss:SSS");

out.println("Id:"+idSessao+"<br/>");
out.println("Data criação:"+sdf.format(dataCriacao)+"<br/>");
out.println("Data último acesso:"+sdf.format(dataUltimoAcesso)+"<br/>");
out.println("Atributos:<br/>");
Enumeration<String> nomesAtributos = sessao.getAttributeNames();
while(nomesAtributos.hasMoreElements()) {
    String nomeAtributo = nomesAtributos.nextElement();
    Object valor = sessao.getAttribute(nomeAtributo);
    out.println(nomeAtributo+"="+valor+"<br/>");
}
```

4.Testar a aplicação
- Criar duas sessões diferentes usando browsers diferentes

```
http://localhost:8084/TestandoEscopoSessao/ArmazenarNaSessao?nome=Daniel
```

- Exibir as duas sessões e mostrar que são diferentes

```
http://localhost:8084/TestandoEscopoSessao/ExibirSessao
```

5. Criar um arquivo web.xml, dentro da pasta WEB-INF, com um timeout de 1 minuto:

```diff
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
	 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
	 version="3.1">
    <session-config>
        <session-timeout>
+            1
        </session-timeout>
    </session-config>
</web-app>
```

6. Testar novamente
7. Fim
