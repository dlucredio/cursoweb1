# Desenvolvimento de Software Para Web 1 - Prof. Daniel Lucrédio
## Instalação de software

### Última atualização: 27/fev/2020

### JDK
---

1. Instalar o Java Development Kit
2. Há duas opções atualmente, da Oracle e OpenJDK

```
https://www.oracle.com/technetwork/java/javase/downloads/index.html

https://openjdk.java.net/
```

3. Os exemplos devem funcionar em qualquer versão, mas o servidor Glassfish 5.1 não funciona com a versão 9 ou superior (apenas a versão 8). Caso só exista as versões mais recentes, deve-se utilizar outro servidor (como WildFly)

4. Para testar se funcionou, executar em um terminal

```sh
java -version
javac -version
```

### Node.js
---

1. Seguir as instruções do site: ```https://nodejs.org/en/```

2. Para testar se funcionou, executar em um terminal

```sh
node --version
npm --version
```

### Apache Maven
---

1. Baixar o Maven de ```https://maven.apache.org```
2. O Maven precisa estar no ```PATH```, e precisa conhecer a variável ```JAVA_HOME```
- Rodar os seguintes comandos no terminal (substituir pelos caminhos corretos):

```sh
export JAVA_HOME=$HOME/Programs/jdk1.8.0_212/
export PATH=$HOME/Programs/apache-maven-3.6.1/bin:$PATH
```

- Esses comandos precisam ser executados sempre que um novo terminal for iniciado. É também possível configurar permanentemente (consulte a documentação do Sistema Operacional para como fazer isso).
- No Linux/Mac, uma boa prática é criar um script temporário para evitar de mexer nas configurações globais
- Criar um arquivo useMaven.sh, com permissão de execução, com o conteúdo acima
- Antes de iniciar o uso do Maven, executar

```sh
source useMaven.sh
```

3. Para testar se funcionou, executar:

```sh
mvn -version
```

4. Para compilar e gerar um arquivo .war, utilizar o seguinte comando

```sh
mvn clean package
```

5. Opcionalmente, é possível criar um Maven Wrapper para que o projeto tenha sua própria instalação do Maven, fixando assim a versão e evitando problemas de versionamento. Também evita a necessidade de executar o comando ```source``` a cada novo terminal.

- Abrir um terminal dentro da pasta do projeto (alo-mundo-web-maven) e executar o seguinte comando

```sh
mvn -N io.takari:maven:wrapper
```

- Depois disso, será criado um arquivo executável ```mvnw```. Ao invés de executar ```mvn```, basta executar esse ```mvnw```.
- Esse arquivo ```mvnw``` e demais criados devem ser compartilhados com a equipe no repositório de controle de versões

### Eclipse Glassfish
---
A versão 5.1 só funciona com JDK 8. Caso só tenha o JDK 9+, utilize o WildFly. A versão 6, a ser lançada em meados de 2020, deve suportar JDK 11

1. Baixar o servidor: ```https://projects.eclipse.org/projects/ee4j.glassfish```

2. Descompactar em alguma pasta
3. Configurar o nome/senha de administrador do Glassfish para deploy automático (default é ```admin```/```<vazio>```)

```sh
./glassfish/bin/asadmin change-admin-password
```

4. Para iniciar/interromper o servidor

```sh
./glassfish/bin/asadmin start-domain
./glassfish/bin/asadmin stop-domain
```

5. Para fazer o deploy pelo Maven, adicionar as seguintes configurações no arquivo pom.xml (substituir nos campos correspondentes o local de instalação do Glassfish, usuário e senha)

```xml
<plugin>
    <groupId>org.glassfish.maven.plugin</groupId>
    <artifactId>maven-glassfish-plugin</artifactId>
    <configuration>
    <glassfishDirectory>/Users/daniellucredio/Programs/glassfish5/glassfish</glassfishDirectory>
    <user>admin</user>
    <adminPassword>admin</adminPassword>
    <domain>
        <name>domain1</name>
        <host>localhost</host>
        <adminPort>4848</adminPort>
    </domain>
    <components>
        <component>
        <name>${project.artifactId}</name>
        <artifact>${project.build.directory}/${project.build.finalName}.war</artifact>
        </component>
    </components>
    </configuration>
    <executions>
    <execution>
        <goals>
        <goal>deploy</goal>
        </goals>
    </execution>
    </executions>
</plugin>
```

6. Executar os seguintes comandos, dentro do diretório do projeto Maven, para instalar, reinstalar e desinstalar (O Glassfish precisa estar rodando)

```sh
mvn clean package glassfish:deploy
mvn clean package glassfish:redeploy
mvn clean package glassfish:undeploy
```

### WildFly
---

1. Baixar o servidor: ```https://wildfly.org/```

2. Seguir os passos de "Getting Started" para iniciar o servidor e configurar um usuário: ```https://docs.wildfly.org/```

- Obs: para os exemplos, recomenda-se iniciar o servidor no modo standalone, executando:

```sh
<diretório WildFly>/bin/standalone.sh
````

- Para interromper, basta digitar CTRL+C

3. Para fazer o deploy pelo Maven, adicionar as seguintes linhas no arquivo pom.xml

```xml
<plugin>
    <groupId>org.wildfly.plugins</groupId>
    <artifactId>wildfly-maven-plugin</artifactId>
    <version>2.0.1.Final</version>
</plugin>
```

4. Executar os seguintes comandos, dentro do diretório do projeto Maven, para instalar, reinstalar e desinstalar (O WildFly precisa estar rodando)

```sh
mvn clean wildfly:deploy
mvn clean wildfly:redeploy
mvn wildfly:undeploy
```

-  Por default, ao executar wildfly:deploy, ele já tenta o redeploy, caso já exista a mesma aplicação implantada

### MongoDB
---

1. Baixar o servidor: ```https://www.mongodb.com/```

2. Descompactar em alguma pasta
3. Para rodar, executar o seguinte comando:

```sh
<pasta instalacao>/bin/mongod --dbpath <pasta para armazenar os bancos>
```

### Apache Derby
---

1. Baixar o servidor: ```http://db.apache.org/derby/```

2. Descompactar o conteúdo em alguma pasta

3. Para iniciar/interromper o servidor, abrir o terminal e executar os seguintes comandos (substituir o caminho do derby.system.home por uma pasta onde serão criados os bancos de dados e a instalação do Derby onde foi descompactado seu conteúdo):

```sh
java -Dderby.system.home=<pasta com bancos de dados> -jar <instalação Derby>/lib/derbyrun.jar server start

java -Dderby.system.home=<pasta com bancos de dados> -jar <instalação Derby>/lib/derbyrun.jar server shutdown
```

- No exemplo a seguir, o servidor está instalado na pasta ```/Users/daniellucredio/Programs/db-derby-10.15.2.0-bin``` e irá buscar os bancos de dados na pasta ```/Users/daniellucredio/DerbyDatabases```:

```sh
java -Dderby.system.home=/Users/daniellucredio/DerbyDatabases -jar /Users/daniellucredio/Programs/db-derby-10.15.2.0-bin/lib/derbyrun.jar server start
```

4. Para criar um banco de dados, executar em um terminal (não é necessário que o servidor esteja rodando):

```sh
java -Dderby.system.home=<caminho bancos de dados> -Dij.protocol=jdbc:derby: -jar <instalação Derby>/lib/derbyrun.jar ij
```

5. Uma vez executando o comando ```ij```, executar o seguinte comando (obs: ```meubanco``` é o nome do banco de dados. ```create=true``` indica que o banco será criado, caso não exista):

```
CONNECT 'meubanco;create=true';
```

- Caso queira criar o banco com um usuário e senha:

```
CONNECT 'meubanco;create=true;user=usuario;password=senha';
```

- Após esse comando, uma pasta chamada "meubanco" será criada no diretório especificado em derby.system.home

- Caso não queira criar um novo banco, apenas conectar, executar o comando:

```
CONNECT 'meubanco';
```

6. Uma vez conectado, é possível executar comandos SQL:

```sql
CREATE TABLE Teste (codigo int, nome varchar(100));
````

7. Para sair do ```ij```, executar:

```
exit;
```

