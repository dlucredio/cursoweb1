# Desenvolvimento de Software para Web 1 - Prof. Daniel Lucrédio
## Módulo 4 - Exemplos

### Demonstração 1 - Usando padrões de projeto Web

1. Criar um novo projeto Web:
- Group Id: ```br.ufscar.dc.web1```
- Artifact Id: ```bolao-da-copa```
- Verificar, no arquivo ```web.xml``` gerado, se a versão do aplicativo é pelo menos ```2.4```. Caso não seja, adicionar:

```diff
+<web-app version="2.4">
  <display-name>Archetype Created Web Application</display-name>
</web-app>
```
- Adicionar as seguintes dependências e plugin ao arquivo ```pom.xml```:


```diff
<project ... >
   ... 
  <dependencies>
    ...
+    <dependency>
+      <groupId>javax.servlet</groupId>
+      <artifactId>javax.servlet-api</artifactId>
+      <version>4.0.0</version>
+      <scope>provided</scope>
+    </dependency>
+    <dependency>
+      <groupId>javax.servlet</groupId>
+      <artifactId>jstl</artifactId>
+      <version>1.2</version>
+      <scope>runtime</scope>
+    </dependency>
+    <dependency>
+      <groupId>commons-beanutils</groupId>
+      <artifactId>commons-beanutils</artifactId>
+      <version>1.9.4</version>
+    </dependency>
+    <dependency>
+      <groupId>javax.annotation</groupId>
+      <artifactId>javax.annotation-api</artifactId>
+      <version>1.3.2</version>
+      <scope>provided</scope>
+    </dependency>
  </dependencies>   
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

2. Criar um novo banco de dados, chamado ```bolao```, com usuario/senha=```bolao```/```bolao```:

- No Apache Derby, basta executar o comando ```ij``` (não é necessário que o banco esteja rodando, basta configurar o ambiente corretamente) e executar:

```
connect 'bolao;create=true;user=bolao;password=bolao';
```

3. Criar as tabelas

```sql
create table Usuario
(
id integer not null generated always as identity (start with 1, increment by 1),
nome varchar(256) not null,
email varchar(256) not null,
telefone varchar(24) not null,
dataDeNascimento date,
CONSTRAINT primary_key PRIMARY KEY (id)
);

create table Palpite
(
id integer not null generated always as identity (start with 1, increment by 1),
campeao varchar(256) not null,
vice varchar(256) not null,
palpiteiro integer references Usuario (id)
);
```

4. Adicionar o recurso JNDI ao Tomcat, no arquivo ```<Instalação Tomcat>/conf/server.xml```

- No caso do Apache Derby, o conteúdo é o seguinte:

```diff
...
    <GlobalNamingResources>
      ...
+      <Resource name="jdbc/BolaoDB" auth="Container" type="javax.sql.DataSource"
+               maxActive="100" maxIdle="30" maxWait="10000"
+               username="bolao" password="bolao" driverClassName="org.apache.derby.jdbc.ClientDriver"
+               url="jdbc:derby://localhost:1527/bolao"/>
    </GlobalNamingResources>
```

5. Adicionar o link ao recurso JNDI no projeto, no arquivo ```webapp/META-INF/context.xml``` (criar pasta/arquivo caso não exista)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context path="/bolao-da-copa">
  <ResourceLink global="jdbc/BolaoDB" name="jdbc/BolaoDBLocal" type="javax.sql.DataSource"/>
</Context>
```

6. Caso ainda não tenha feito, copiar os JDBC Drivers para a instalação do Tomcat

- No caso do Apache Derby, basta copiar todos os jars da pasta lib de sua instalação para a instalação do Tomcat (pasta lib também)

```sh
cp <Instalação Derby>/lib/*.jar <Instalação Tomcat>/lib 
```

7. Iniciar os servidores (Web e SGBD)

8. Criar os Value Objects (pacote ```br.ufscar.dc.web1.bolao.beans```)

- Usuario

```java
package br.ufscar.dc.web1.bolao.beans;

import java.util.Date;

public class Usuario {
    private int id;
    private String nome, email, telefone;
    private Date dataDeNascimento;
    ... setters e getters
}
```

- Palpite

```java
package br.ufscar.dc.web1.bolao.beans;

public class Palpite {
    private int id;
    private String campeao;
    private String vice;
    private Usuario palpiteiro;
    ... setters e getters
}
```

9. Criar os DAOs (pacote ```br.ufscar.dc.web1.bolao.dao```)

- DAO Usuário
```java
package br.ufscar.dc.web1.bolao.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Date;

import javax.naming.NamingException;
import javax.sql.DataSource;

import br.ufscar.dc.web1.bolao.beans.Usuario;

public class UsuarioDAO {

    private final static String CRIAR_USUARIO_SQL = "insert into Usuario"
            + " (nome, email, telefone, dataDeNascimento)"
            + " values (?,?,?,?)";

    private final static String BUSCAR_USUARIO_SQL = "select"
            + " id, nome, email, telefone, dataDeNascimento"
            + " from usuario"
            + " where id=?";
    
    DataSource dataSource;

    public UsuarioDAO(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    
    public Usuario gravarUsuario(Usuario u) throws SQLException, NamingException {
        try (Connection con = dataSource.getConnection();
                PreparedStatement ps = con.prepareStatement(CRIAR_USUARIO_SQL, Statement.RETURN_GENERATED_KEYS);) {
            ps.setString(1, u.getNome());
            ps.setString(2, u.getEmail());
            ps.setString(3, u.getTelefone());
            ps.setDate(4, new java.sql.Date(u.getDataDeNascimento().getTime()));
            ps.execute();

            try (ResultSet rs = ps.getGeneratedKeys()) {
                rs.next();
                u.setId(rs.getInt(1));
            }
        }
        return u;
    }

    public Usuario buscarUsuario(int id) throws SQLException, NamingException {
        try (Connection con = dataSource.getConnection();
                PreparedStatement ps = con.prepareStatement(BUSCAR_USUARIO_SQL)) {
            ps.setInt(1, id);

            try (ResultSet rs = ps.executeQuery()) {
                rs.next();
                Usuario u = new Usuario();
                u.setId(rs.getInt("id"));
                u.setNome(rs.getString("nome"));
                u.setEmail(rs.getString("email"));
                u.setTelefone(rs.getString("telefone"));
                u.setDataDeNascimento(new Date(rs.getDate("dataDeNascimento").getTime()));
                return u;
            }
        }
    }
}
```

- DAO Palpite

```java
package br.ufscar.dc.web1.bolao.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

import javax.naming.NamingException;
import javax.sql.DataSource;

import br.ufscar.dc.web1.bolao.beans.Palpite;
import br.ufscar.dc.web1.bolao.beans.Usuario;

public class PalpiteDAO {

    private final static String CRIAR_PALPITE_SQL = "insert into Palpite"
            + " (campeao, vice, palpiteiro)"
            + " values (?,?,?)";

    private final static String LISTAR_PALPITES_SQL = "select"
            + " p.id as palpiteId, p.campeao, p.vice,"
            + " u.id as usuarioId, u.nome, u.email, u.telefone, u.dataDeNascimento"
            + " from Palpite p inner join Usuario u on p.palpiteiro = u.id";

    private final static String LISTAR_PALPITES_POR_SELECAO_SQL = "select"
            + " p.id as palpiteId, p.campeao, p.vice,"
            + " u.id as usuarioId, u.nome, u.email, u.telefone, u.dataDeNascimento"
            + " from Palpite p inner join Usuario u on p.palpiteiro = u.id"
            + " where campeao = ? or vice = ?";

    DataSource dataSource;

    public PalpiteDAO(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public Palpite gravarPalpite(Palpite p) throws SQLException, NamingException {
        try (Connection con = dataSource.getConnection();
                PreparedStatement ps = con.prepareStatement(CRIAR_PALPITE_SQL, Statement.RETURN_GENERATED_KEYS);) {
            ps.setString(1, p.getCampeao());
            ps.setString(2, p.getVice());
            ps.setInt(3, p.getPalpiteiro().getId());
            ps.execute();

            try (ResultSet rs = ps.getGeneratedKeys()) {
                rs.next();
                p.setId(rs.getInt(1));
            }
        }
        return p;
    }

    public List<Palpite> listarTodosPalpites() throws SQLException, NamingException {
        List<Palpite> ret = new ArrayList<>();
        try (Connection con = dataSource.getConnection();
                PreparedStatement ps = con.prepareStatement(LISTAR_PALPITES_SQL)) {

            try (ResultSet rs = ps.executeQuery()) {
                while (rs.next()) {
                    Palpite p = new Palpite();
                    Usuario u = new Usuario();
                    p.setId(rs.getInt("palpiteId"));
                    p.setCampeao(rs.getString("campeao"));
                    p.setVice(rs.getString("vice"));
                    u.setId(rs.getInt("usuarioId"));
                    u.setNome(rs.getString("nome"));
                    u.setEmail(rs.getString("email"));
                    u.setTelefone(rs.getString("telefone"));
                    u.setDataDeNascimento(new Date(rs.getDate("dataDeNascimento").getTime()));
                    p.setPalpiteiro(u);
                    ret.add(p);
                }
            }
        }
        return ret;
    }

    public List<Palpite> listarTodosPalpitesPorSelecao(String selecao) throws SQLException {
        List<Palpite> ret = new ArrayList<>();
        try (Connection con = dataSource.getConnection();
                PreparedStatement ps = con.prepareStatement(LISTAR_PALPITES_POR_SELECAO_SQL)) {

            ps.setString(1, selecao);
            ps.setString(2, selecao);
            try (ResultSet rs = ps.executeQuery()) {
                while (rs.next()) {
                    Palpite p = new Palpite();
                    Usuario u = new Usuario();
                    p.setId(rs.getInt("palpiteId"));
                    p.setCampeao(rs.getString("campeao"));
                    p.setVice(rs.getString("vice"));
                    u.setId(rs.getInt("usuarioId"));
                    u.setNome(rs.getString("nome"));
                    u.setEmail(rs.getString("email"));
                    u.setTelefone(rs.getString("telefone"));
                    u.setDataDeNascimento(new Date(rs.getDate("dataDeNascimento").getTime()));
                    p.setPalpiteiro(u);
                    ret.add(p);                }
            }
        }
        return ret;
    }
}
```

10. Criar um arquivo ```webapp/index.jsp```

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Bolão da copa</title>
        <link rel="stylesheet" type="text/css" href="estilo.css" />        
    </head>
    <body>
        <h1>Bem-vindo ao bolão da Copa 2010!</h1>
        <hr>
        <p>Este é o bolão da Copa</p>
        <p>Escolha o que deseja fazer:</p>
        <a href="#">Fazer um palpite</a><br/>
        <a href="#">Ver todos os palpites</a><br/>
        <a href="#">Ver todos os palpites envolvendo o Brasil</a><br/>
    </body>
</html>
```

11. Criar o arquivo ```webapp/estilo.css```

```css
body { background-color: #99EF99;}
h1 { font-style: italic;}
p { color: maroon; }
a {color: black; font-variant: small-caps; font-size: 1.5em;}
table { border-collapse:collapse; }
table, th, td { border: 1px solid black; }
th { background-color:green; color:yellow; }
td { text-align: center; }
td.esquerda { text-align: left; }
ul.erro { color: red; }
```

12. Testar e mostrar o que já tem: ```http://localhost:8080/bolao-da-copa/index.jsp```

13. Modificar o arquivo index.jsp, para adicionar o link para fazer um palpite

```diff
        <p>Escolha o que deseja fazer:</p>
+        <a href="palpiteForm.jsp">Fazer um palpite</a><br/>
        <a href="#">Ver todos os palpites</a><br/>
        <a href="#">Ver todos os palpites envolvendo o Brasil</a><br/>
```

14. Criar o arquivo ```webapp/palpiteForm.jsp```

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Bolão da Copa</title>
        <link rel="stylesheet" type="text/css" href="estilo.css" />
    </head>
    <body>
        <h1>Novo palpite</h1>
        <hr>

        <form action="NovoPalpiteServlet" method="post">
            Digite seus dados:<br/>
            Nome: <input name="nome" type="text" value="" /><br/>
            E-mail: <input name="email" type="text" value="" /><br/>
            Telefone: <input name="telefone" type="text" value="" /><br/>
            Data de nascimento: <input name="dataDeNascimento" type="text" value="" /><br/>
            Campeão: <input name="campeao" type="text" value="" /><br/>
            Vice: <input name="vice" type="text" value="" /><br/>
            <input type="submit" value="Enviar"/>
        </form>
    </body>
</html>
```

15. Criar o bean do formulário

```java
package br.ufscar.dc.web1.bolao.forms;

public class NovoPalpiteFormBean {

    private String nome, email, telefone, dataDeNascimento, campeao, vice;

    ... setters e getters
}
```

16. A seguir faremos uso da biblioteca Apache Commons BeanUtils, adicionada como dependência na criação do projeto

17. Criar o controlador NovoPalpiteServlet

```java
package br.ufscar.dc.web1.bolao.servlets;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.beanutils.BeanUtils;

import br.ufscar.dc.web1.bolao.forms.NovoPalpiteFormBean;

@WebServlet(name = "NovoPalpiteServlet", urlPatterns = { "/NovoPalpiteServlet" })
public class NovoPalpiteServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    protected void processRequest(HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {
        request.setCharacterEncoding("UTF-8");
        NovoPalpiteFormBean npfb = new NovoPalpiteFormBean();
        try {
            // Obs: BeanUtils é uma classe da biblioteca
            // Apache Commons BeanUtils
            // http://commons.apache.org/beanutils/
            BeanUtils.populate(npfb, request.getParameterMap());
            request.getSession().setAttribute("novoPalpite", npfb);
            request.getRequestDispatcher("confirmarPalpite.jsp").forward(request, response);
        } catch (Exception e) {
            request.setAttribute("mensagem", e.getLocalizedMessage());
            request.getRequestDispatcher("erro.jsp").forward(request, response);
        }
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        processRequest(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        processRequest(req, resp);
    }
}
```

18. Criar página ```webapp/erro.jsp```

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Bolão da copa</title>
        <link rel="stylesheet" type="text/css" href="estilo.css" />
    </head>
    <body>
        <h1>Opa, parece que ocorreu um problema</h1>
        Descrição: ${mensagem}
    </body>
</html>
```

19. Testar para ver o erro, modificar o servlet

```diff
        try {
            // Obs: BeanUtils é uma classe da biblioteca
            // Jakarta Commons BeanUtils
            // http://commons.apache.org/beanutils/
            BeanUtils.populate(npfb, request.getParameterMap());
+            int a = 0;
+            int b = 2/a;
            request.getSession().setAttribute("novoPalpite", npfb);
            request.getRequestDispatcher("confirmarPalpite.jsp").forward(request, response);
        } catch (Exception e) {
            request.setAttribute("mensagem", e.getLocalizedMessage());
            request.getRequestDispatcher("erro.jsp").forward(request, response);
        }
```

20. Corrigir o erro anterior, e criar página ```webapp/confirmarPalpite.jsp```

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Bolão da Copa</title>
        <link rel="stylesheet" type="text/css" href="estilo.css" />
    </head>
    <body>
        <h1>Novo palpite</h1>
        Atenção! Deseja realmente enviar seu palpite?
        <br/>
        Uma vez enviado, você está automaticamente aceitando
        pagar R$ 20,00 de inscrição.
        <br/><br/>
        Nome: ${sessionScope.novoPalpite.nome}<br/>
        E-mail: ${sessionScope.novoPalpite.email}<br/>
        Telefone: ${sessionScope.novoPalpite.telefone}<br/>
        Data de nascimento: ${sessionScope.novoPalpite.dataDeNascimento}<br/>
        Campeão: ${sessionScope.novoPalpite.campeao}<br/>
        Vice: ${sessionScope.novoPalpite.vice}<br/>
        <br/>
        <a href="GravarPalpiteServlet">Confirmar</a>
        <a href="palpiteForm.jsp">Modificar</a>
        <a href="index.jsp">Cancelar</a>
    </body>
</html>
```

21. Mostrar rodando. Testar a opção “Modificar” e mostrar que não aparecem os campos
22. Alterar ```webapp/palpiteForm.jsp```

```diff
Nome: <input name="nome" type="text" value="${sessionScope.novoPalpite.nome}" /><br/>
E-mail: <input name="email" type="text" value="${sessionScope.novoPalpite.email}" /><br/>
Telefone: <input name="telefone" type="text" value="${sessionScope.novoPalpite.telefone}" /><br/>
Data de nascimento: <input name="dataDeNascimento" type="text" value="${sessionScope.novoPalpite.dataDeNascimento}" /><br/>
Campeão: <input name="campeao" type="text" value="${sessionScope.novoPalpite.campeao}" /><br/>
Vice: <input name="vice" type="text" value="${sessionScope.novoPalpite.vice}" /><br/>
````

23. Alterar também ```webapp/index.jsp```

```diff
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE html>
+ <%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
+ <c:remove scope="session" var="novoPalpite" />
<html>
...
```

24. Testar novamente e mostrar que ao clicar em “Modificar”, os campos agora aparecem preenchidos. E também, ao abrir a aplicação em index.jsp, os campos não são preenchidos com o último palpite.

25. Fazer a validação
- Modificar a classe ```NovoPalpiteFormBean```, incluindo um método de validação

```java
public List<String> validar() {
    List<String> mensagens = new ArrayList<String>();
    if (nome.trim().length() == 0) {
        mensagens.add("Nome não pode ser vazio!");
    }
    if (!email.contains("@")) {
        mensagens.add("E-mail está em formato incorreto!");
    }
    try {
        SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy");
        sdf.parse(dataDeNascimento);
    } catch (ParseException pe) {
        mensagens.add("Data de nascimento deve estar no formato dd/mm/aaaa!");
    }
    if (campeao.equalsIgnoreCase(vice)) {
        mensagens.add("Campeão e vice devem ser diferentes!");
    }
    return (mensagens.isEmpty() ? null : mensagens);
}
```

- Modificar o controlador ```NovoPalpiteServlet```, para incluir a lógica de validação

```diff
BeanUtils.populate(npfb, request.getParameterMap());
request.getSession().setAttribute("novoPalpite", npfb);
+ List<String> mensagens = npfb.validar();
+ if (mensagens == null) {
+     request.getRequestDispatcher("confirmarPalpite.jsp").forward(request, response);
+ } else {
+     request.setAttribute("mensagens", mensagens);
+     request.getRequestDispatcher("palpiteForm.jsp").forward(request, response);
+ }
- request.getRequestDispatcher("confirmarPalpite.jsp").forward(request, response);
```

- Modificar ```webapp/palpiteForm.jsp```

```diff
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE html>
+ <%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Bolão da Copa</title>
        <link rel="stylesheet" type="text/css" href="estilo.css" />
    </head>
    <body>
        <h1>Novo palpite</h1>
        <hr>
+        <c:if test="${!empty requestScope.mensagens}">
+            <ul class="erro">
+            <c:forEach items="${requestScope.mensagens}" var="mensagem">
+                <li>${mensagem}</li>
+            </c:forEach>
+            </ul>
+            <hr>
+        </c:if>

        <form action="NovoPalpiteServlet" method="post">
        ...
```

26. Testar a validação
27. Criar o controlador ```GravarPalpiteServlet```

```java
package br.ufscar.dc.web1.bolao.servlets;

import java.io.IOException;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

import javax.annotation.Resource;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.sql.DataSource;

import br.ufscar.dc.web1.bolao.beans.Palpite;
import br.ufscar.dc.web1.bolao.beans.Usuario;
import br.ufscar.dc.web1.bolao.dao.PalpiteDAO;
import br.ufscar.dc.web1.bolao.dao.UsuarioDAO;
import br.ufscar.dc.web1.bolao.forms.NovoPalpiteFormBean;

@WebServlet(name = "GravarPalpiteServlet", urlPatterns = { "/GravarPalpiteServlet" })
public class GravarPalpiteServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    @Resource(name = "jdbc/BolaoDBLocal")
    DataSource dataSource;

    protected void processRequest(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        NovoPalpiteFormBean npfb = (NovoPalpiteFormBean) request.getSession().getAttribute("novoPalpite");
        request.getSession().removeAttribute("novoPalpite");

        UsuarioDAO udao = new UsuarioDAO(dataSource);
        PalpiteDAO pdao = new PalpiteDAO(dataSource);

        SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy");
        Date dataNascimento = null;
        try {
            dataNascimento = sdf.parse(npfb.getDataDeNascimento());
        } catch (ParseException e) {
            request.setAttribute("mensagem", e.getLocalizedMessage());
            request.getRequestDispatcher("erro.jsp").forward(request, response);
        }
        try {
            Usuario u = new Usuario();
            u.setNome(npfb.getNome());
            u.setEmail(npfb.getEmail());
            u.setTelefone(npfb.getTelefone());
            u.setDataDeNascimento(dataNascimento);
            u = udao.gravarUsuario(u);
            Palpite p = new Palpite();
            p.setCampeao(npfb.getCampeao());
            p.setVice(npfb.getVice());
            p.setPalpiteiro(u);
            p = pdao.gravarPalpite(p);
            request.setAttribute("mensagem", "Obrigado pelo palpite!");
            request.getRequestDispatcher("index.jsp").forward(request, response);
        } catch (Exception e) {
            e.printStackTrace();
            request.setAttribute("mensagem", e.getLocalizedMessage());
            request.getRequestDispatcher("erro.jsp").forward(request, response);
        }
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        processRequest(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        processRequest(req, resp);
    }
}
```

28. Modificar o arquivo ```webapp/index.jsp```, para incluir a mensagem

```diff
    <body>
        <h1>Bem-vindo ao bolão da Copa 2010!</h1>
        <hr>
+        <c:if test="${!empty mensagem}">
+            ${mensagem}
+            <hr>
+        </c:if>
        <p>Este é o bolão da Copa</p>
```

29. Testar a aplicação
30. Criar controlador VerPalpitesServlet

```java
package br.ufscar.dc.web1.bolao.servlets;

import java.io.IOException;
import java.util.List;

import javax.annotation.Resource;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.sql.DataSource;

import br.ufscar.dc.web1.bolao.beans.Palpite;
import br.ufscar.dc.web1.bolao.dao.PalpiteDAO;

@WebServlet(name = "VerPalpitesServlet", urlPatterns = { "/VerPalpitesServlet" })
public class VerPalpitesServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    @Resource(name = "jdbc/BolaoDBLocal")
    DataSource dataSource;

    protected void processRequest(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        // try {
        //     Context ctx = new InitialContext();
        //     dataSource = (DataSource) ctx.lookup("java:comp/env/jdbc/BolaoDBLocal");
        // } catch (Exception e) {
        //     e.printStackTrace();
        // }
        PalpiteDAO pdao = new PalpiteDAO(dataSource);
        String selecao = request.getParameter("selecao");
        List<Palpite> todosPalpites = null;
        try {
            if (selecao == null) {
                todosPalpites = pdao.listarTodosPalpites();
            } else {
                todosPalpites = pdao.listarTodosPalpitesPorSelecao(selecao);
            }
            request.setAttribute("listaPalpites", todosPalpites);
            request.getRequestDispatcher("listaPalpites.jsp").forward(request, response);
        } catch (Exception e) {
            e.printStackTrace();
            request.setAttribute("mensagem", e.getLocalizedMessage());
            request.getRequestDispatcher("erro.jsp").forward(request, response);
        }

    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        processRequest(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        processRequest(req, resp);
    }
}
```

31. Criar página ```webapp/listaPalpites.jsp```

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Bolão da copa</title>
        <link rel="stylesheet" type="text/css" href="estilo.css" />
    </head>
    <body>
        <h1>Palpites da copa</h1>
        <hr>
        <c:if test="${empty requestScope.listaPalpites}">
            Não há palpites!
        </c:if>
        <c:if test="${!empty requestScope.listaPalpites}">
            <table>
                <tr>
                    <th class="esquerda">Usuário</th>
                    <th>Campeão</th>
                    <th>Vice</th>
                </tr>
                <c:forEach items="${requestScope.listaPalpites}" var="palpite">
                    <tr>
                        <td class="esquerda">${palpite.palpiteiro.nome}</td>
                        <td>${palpite.campeao}</td>
                        <td>${palpite.vice}</td>
                    </tr>
                </c:forEach>
            </table>
        </c:if>
    </body>
</html>
```

32. Modificar o arquivo index.jsp para incluir os links

```diff
<p>Escolha o que deseja fazer:</p>
<a href="palpiteForm.jsp">Fazer um palpite</a><br/>
+ <a href="VerPalpitesServlet">Ver todos os palpites</a><br/>
+ <a href="VerPalpitesServlet?selecao=Brasil">Ver todos os palpites envolvendo o Brasil</a><br/>
```
33. Testar
34. Fim


## Exercícios de fixação (em ordem crescente de dificuldade)

1. Adicionar uma opção para que o usuário possa pesquisar palpites por qualquer seleção (e não apenas o Brasil)

2. Adicionar uma opção para mostrar todos os palpites de um determinado usuário

3. Modificar a aplicação para que só seja possível adicionar seleções de uma lista pré-definida (que está armazenada no banco de dados)

4. Adicionar mais duas opções ao palpite (terceiro e quarto lugares, além de campeão e vice). Em seguida, adicionar uma função onde é informado o resultado da Copa, e a aplicação mostra quem ganhou o bolão, com base em alguma fórmula de sua escolha

5. Adicionar login e senha para uso do sistema, com papeis específicos aos usuários, como: usuário comum só pode fazer palpites, administrador pode acessar as demais funções

