# Desenvolvimento de Software para Web 1 - Prof. Daniel Lucrédio
## Módulo 3 - Exemplos

### Demonstração 1 - Aplicação “Alo mundo” usando JSP
---

1. Criar um novo projeto web:
- Group Id: ```br.ufscar.dc.web1```
- Artifact Id: ```alomundo-jsp```
2. Apagar o arquivo index.jsp que já é criado por default (em versões novas isso não existe mais)
3. Criar um servlet alo mundo qualquer
4. Criar um JSP alo mundo qualquer:

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>JSP Page</title>
    </head>
    <body>
        <h1>Alo Mundo com JSP</h1>
    </body>
</html>
```

5. Executar o projeto e abrir o JSP
6. Exibir o servlet gerado, dentro da pasta work do TomCat
7. Adicionar um código no servlet, para fazer um for de 1 a 10

```java
for(int i=0;i<10;i++) {
    String linha = "Linha "+i;
    out.println(i+": "+linha+"<br/>");
}
```

7. Adicionar um código equivalente no JSP, para comparar

```jsp
<% for(int i=0;i<10;i++) {
    String linha = "Linha "+i;
    %>
        <%=i%>: <%=linha%> <br/>
<% } %>
```

8. Rodar e ver o efeito
9. Mostrar o servlet gerado
10. Fim

### Demonstração 2 - Diferentes tipos de erros com JSP
---

1. Criar um novo projeto web:
- Group Id: ```br.ufscar.dc.web1```
- Artifact Id: ```erros-jsp```
2. Criar um arquivo ```index.jsp```, e introduzir um erro na diretiva ```<%@page ...``` (Ex: ```<%#paje ...```)

```jsp
<%#paje contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>JSP Page</title>
    </head>
    <body>
        <h1>Alo Mundo com JSP</h1>
    </body>
</html>
```

3. Executar e mostrar que dá erro, e nem gera o servlet
4. Modificar o arquivo index.jsp, corrigindo o erro introduzido em 2
5. Modificar o arquivo index.jsp, introduzindo um erro de compilação

```jsp
<% int a = 2 %>
```

6. Executar e mostrar que dá erro, mas gera o servlet
- Mostrar o servlet e achar o erro
7. Corrigir o erro introduzido em 5, e adicionar código com erro de execução

```jsp
<% int a = 2;
String str = null;
a = str.length();
%>
```

8. Executar e mostrar que dá erro, gerando o servlet
- Mostrar como rastrear o erro até o JSP. Primeiro, mostrar a saída do Tomcat, e falar que ele aponta a linha do servlet
- Mostrar que às vezes o próprio Tomcat indica o erro no JSP. Mas em muitos casos é preciso rastrear na mão até o JSP
9. Fim

### Demonstração 3 - Objetos implícitos
---

1. Criar um novo projeto web:
- Group Id: ```br.ufscar.dc.web1```
- Artifact Id: ```alomundo-objetos-implicitos```
2. Adicionar o seguinte código em um JSP:

```jsp
<%
int num = Integer.parseInt(request.getParameter("num"));
String nome = request.getParameter("nome");
for(int i=0;i<num;i++) {
%>
Olá <%=nome%>!<br/>
<% } %>
```

3. Testar com diferentes valores de “num” e “nome”
4. Fim

### - Demonstração 4 - Transferindo controle para outros recursos web
---

1. Criar um novo projeto web: 
- Group Id: ```br.ufscar.dc.web1```
- Artifact Id: ```transferencia-controle-jsp```

2. Adicionar um formulário para login em index.jsp

```jsp
    <body>
        Login<br/>
        <form action="login.jsp">
            Usuário: <input type="text" name="usuario" /><br/>
            Senha: <input type="password" name="senha" /><br/>
            <input type="submit" value="Login" />
        </form>
    </body>
```

3. Criar arquivo ```login.jsp```, com a lógica de controle de login

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<%
String usuario = request.getParameter("usuario");
String senha = request.getParameter("senha");
if(usuario.equalsIgnoreCase(senha)) {
%>
<jsp:forward page="sucesso.jsp">
    <jsp:param name="nomeCompleto" value="Daniel Lucrédio" />
    <jsp:param name="ultimoAcesso" value="10/02/2010" />
</jsp:forward>
<% }
else { %>
<jsp:forward page="index.jsp" />
<% } %>
```

4. Criar a página ```sucesso.jsp```

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
   "http://www.w3.org/TR/html4/loose.dtd">

<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>JSP Page</title>
    </head>
    <body>
        Bem-vindo <%=request.getParameter("nomeCompleto")%>!
        <br/>
        Seu último acesso foi em <%=request.getParameter("ultimoAcesso")%>!
    </body>
</html>
```

5. Testar a aplicação
6. Mostrar o código gerado para login.jsp, identificar a linha do forward, e como os parâmetros são adicionados ao request
7. Criar a página ```cabecalho.jsp```

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
Bem-vindo <%=request.getParameter("nomeCompleto")%>!
<br/>
Seu último acesso foi em <%=request.getParameter("ultimoAcesso")%>!
<hr>
```

8. Modificar a página sucesso.jsp para incluir o cabecalho.jsp

```
<jsp:include page="cabecalho.jsp" />
Menu de opções:<br/><br/>
Conteúdo da página<br/>
Conteúdo da página<br/>
Conteúdo da página<br/>
Conteúdo da página<br/>
Conteúdo da página<br/>
Conteúdo da página<br/>
```

9. Testar a aplicação
10. Mostrar o código gerado para sucesso.jsp, e identificar a linha do include
11. Modificar sucesso.jsp para usar ```<%@include%>```

```jsp
<%@include file="cabecalho.jsp" %>
```

12. Mostrar o código gerado agora, e comparar
13. Adicionar, na página sucesso.jsp, o seguinte código

```jsp
<% String voceEstaEm = "Home --> Bem-vindo"; %>
```

14. Modificar a página cabecalho.jsp para acessar a variável local

```jsp
Você está em: <%=voceEstaEm%>!
```

15. Testar
16. Fim

### Demonstração 5 - Expression Language
---

1. Criar um novo projeto web:
- Group Id: ```br.ufscar.dc.web1```
- Artifact Id: ```demo-el```
- Verificar, no arquivo ```web.xml``` gerado, se a versão do aplicativo é pelo menos ```2.4```. Caso não seja, adicionar:

```diff
+<web-app version="2.4">
  <display-name>Archetype Created Web Application</display-name>
</web-app>
```
2. Criar uma classe ```br.ufscar.dc.web1.demo.el.beans.Usuario```, com o seguinte código

```java
package br.ufscar.dc.web1.demo.el.beans;

import java.text.SimpleDateFormat;
import java.util.Date;

public class Usuario {
    private String nomeLogin, nome, senha;
    private final Date ultimoAcesso;

    public Usuario() {
        ultimoAcesso = new Date();
    }

    public void setNomeLogin(String nomeLogin) {
        this.nomeLogin = nomeLogin;
    }

    public String getNomeLogin() {
        return nomeLogin;
    }

    public String getNome() {
        return nome;
    }

    public void setNome(String nome) {
        this.nome = nome;
    }

    public String getSenha() {
        return senha;
    }

    public void setSenha(String senha) {
        this.senha = senha;
    }

    public String getUltimoAcesso() {
        SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy - hh:mm:ss:SSS");
        return sdf.format(ultimoAcesso);
    }
}
```

3. Criar uma página ```index.jsp```

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>JSP Page</title>
    </head>
    <body>
        Login<br/>
        <form action="login.jsp">
            Usuário: <input type="text" name="usuario" /><br/>
            Senha: <input type="password" name="senha" /><br/>
            <input type="submit" value="Login" />
        </form>
    </body>

</html>
```

4. Criar uma página ```login.jsp```

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>JSP Page</title>
    </head>
    <%@page import="br.ufscar.dc.web1.demo.el.beans.Usuario" %>
    <%
        String nomeLogin = request.getParameter("usuario");
        String senha = request.getParameter("senha");
        if (senha.equals(nomeLogin)) {
            Usuario usuario = new Usuario();
            usuario.setNome("Daniel Lucrédio");
            usuario.setNomeLogin(nomeLogin);
            usuario.setSenha(senha);
            session.setAttribute("usuarioLogado", usuario);
    %>
    <jsp:forward page="principal.jsp"/>
    <% } else { %>
    <jsp:forward page="index.jsp" />
    <% }%>
</html>
```

5. Criar uma página ```principal.jsp```

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE html>

<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Principal</title>
    </head>
    <body>
        Bem-vindo ${sessionScope.usuarioLogado.nome}
        
        (${sessionScope.usuarioLogado.nomeLogin})!
        <br/>
        Seu último acesso foi em ${sessionScope.usuarioLogado.ultimoAcesso}!<br/>
        Sua senha é ${sessionScope.usuarioLogado.senha}

    </body>
</html>
```

6. Testar
7. Fim


### Demonstração 6 - Demonstração de JSTL

1. Abrir projeto da demonstração 5 (Expression Language)
2. Adicionar biblioteca JSTL via Maven, adicionando as seguintes linhas ao arquivo pom.xml:

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
    <scope>runtime</scope>
</dependency>   
```

3. Modificar o arquivo principal.jsp

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
    "http://www.w3.org/TR/html4/loose.dtd">
<%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<c:set var="local" value="Principal" scope="page" />
<c:set var="acessos" scope="page">
    <table border="1">
        <tr>
            <td>Acessos: 34552</td>
        </tr>
    </table>
</c:set>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Principal</title>
    </head>
    <body>
        Bem-vindo ${sessionScope.usuarioLogado.nome}
        (${sessionScope.usuarioLogado.nomeLogin})!
        <br/>
        Seu último acesso foi em ${sessionScope.usuarioLogado.ultimoAcesso}!<br/>
        Sua senha é ${sessionScope.usuarioLogado.senha}
        <hr>
        Você está em: ${pageScope.local}
        <hr>
        Menu de opções:<br/><br/>
        Conteúdo da página<br/>
        Conteúdo da página<br/>
        Conteúdo da página<br/>
        Conteúdo da página<br/>
        Conteúdo da página<br/>
        Conteúdo da página<br/>

        <hr>
        ${pageScope.acessos}
    </body>
</html>
```

4. Modificar arquivo login.jsp para remover os scriptlets

```jsp
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<c:choose>
    <c:when test="${param.usuario == param.senha}">
        <jsp:useBean id="usuarioLogado" class="br.ufscar.dc.web1.demo.el.beans.Usuario" scope="session" />
        <jsp:setProperty name="usuarioLogado" property="nome" value="Daniel Lucrédio" />
        <jsp:setProperty name="usuarioLogado" property="nomeLogin" param="usuario" />
        <jsp:setProperty name="usuarioLogado" property="senha" value="${param.senha}"/>
        <jsp:forward page="principal.jsp"/>
    </c:when>
    <c:otherwise>
        <jsp:forward page="index.jsp" />
    </c:otherwise>
</c:choose>
```
- Testar
5. Caso propriedade e parâmetro tenham o mesmo nome, é possível omitir o valor

```diff
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<c:choose>
    <c:when test="${param.usuario == param.senha}">
        <jsp:useBean id="usuarioLogado" class="br.ufscar.dc.web1.demo.el.beans.Usuario" scope="session" />
        <jsp:setProperty name="usuarioLogado" property="nome" value="Daniel Lucrédio" />
        <jsp:setProperty name="usuarioLogado" property="nomeLogin" param="usuario" />
+        <jsp:setProperty name="usuarioLogado" property="senha"/>
        <jsp:forward page="principal.jsp"/>
    </c:when>
    <c:otherwise>
        <jsp:forward page="index.jsp" />
    </c:otherwise>
</c:choose>
```

6. Criar uma classe ```br.ufscar.dc.web1.demo.el.beans.ItemMenu```, com duas propriedades String: nome e link

```java
package br.ufscar.dc.web1.demo.el.beans;

public class ItemMenu {
    private String nome, link;

    public ItemMenu() {
    }

    public ItemMenu(String nome, String link) {
        this.nome = nome;
        this.link = link;
    }

    public String getNome() {
        return nome;
    }

    public String getLink() {
        return link;
    }

    public void setNome(String nome) {
        this.nome = nome;
    }

    public void setLink(String link) {
        this.link = link;
    }
}

```

7. Criar uma classe ```br.ufscar.dc.web1.demo.el.beans.Menu```:

```java
package br.ufscar.dc.web1.demo.el.beans;

import java.util.ArrayList;
import java.util.List;

public class Menu {
    List<ItemMenu> itensMenu;

    public Menu() {
        itensMenu = new ArrayList<>();
        itensMenu.add(new ItemMenu("Principal","principal.jsp"));
        itensMenu.add(new ItemMenu("Notícias","noticias.jsp"));
        itensMenu.add(new ItemMenu("Produtos","produtos.jsp"));
        itensMenu.add(new ItemMenu("Fale conosco","contato.jsp"));
    }

    public List<ItemMenu> getItensMenu() {
        return itensMenu;
    }
}
```

8. Modificar o arquivo principal.jsp para implementar um menu dinâmico:

```jsp
Menu de opções:<br/><br/>
<jsp:useBean id="menu" class="br.ufscar.dc.web1.demo.el.beans.Menu" scope="application" />
<c:forEach var="im" items="${menu.itensMenu}">
    <a href="${im.link}">${im.nome}</a><br/>
</c:forEach>
```

9. Testar a aplicação
10. Fim
