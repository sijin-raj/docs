---
title: GitHub Packagesで利用するために Apache Maven を設定する
intro: '{% data variables.product.prodname_registry %} にパッケージを公開するよう Apache Mavenを設定し、{% data variables.product.prodname_registry %} に保存されたパッケージを依存関係としてJavaプロジェクトで利用できます。'
product: '{% data reusables.gated-features.packages %}'
redirect_from:
  - /articles/configuring-apache-maven-for-use-with-github-package-registry
  - /github/managing-packages-with-github-package-registry/configuring-apache-maven-for-use-with-github-package-registry
  - /github/managing-packages-with-github-packages/configuring-apache-maven-for-use-with-github-packages
versions:
  free-pro-team: '*'
  enterprise-server: '>=2.22'
---

{% data reusables.package_registry.packages-ghes-release-stage %}

**ノート:** dockerイメージをインストールしたり公開したりする際に、現時点で{% data variables.product.prodname_registry %}はWindowsイメージのような外部レイヤーはサポートしていません。

### {% data variables.product.prodname_registry %} への認証を行う

{% data reusables.package_registry.authenticate-packages %}

#### 個人アクセストークンでの認証

{% data reusables.package_registry.required-scopes %}

*~/.m2/settings.xml*ファイルを編集して個人アクセストークンを含めることで、Apache Mavenで{% data variables.product.prodname_registry %}の認証を受けられます。 *~/.m2/settings.xml*ファイルがないなら新しく作成してください。

`servers`タグの中に、子として`server`タグを`id`付きで追加し、*USERNAME*を{% data variables.product.prodname_dotcom %}のユーザ名で、*TOKEN*を個人アクセストークンで置き換えてください。

`repositories`の中で、リポジトリの`id`をクレデンシャルを含む`server`タグに追加した`id`にマッピングして、リポジトリを設定してください。 `repositories`の中で、リポジトリの`id`をクレデンシャルを含む`server`タグに追加した`id`にマッピングして、リポジトリを設定してください。 {% data reusables.package_registry.lowercase-name-field %}

複数のリポジトリとやりとりをしたい場合には、それぞれのリポジトリを`repositories`タグの子の個別の`repository`に追加し、それぞれの`id`を`servers` タグのクレデンシャルにマッピングできます。

{% data reusables.package_registry.apache-maven-snapshot-versions-supported %}

{% if currentVersion != "free-pro-team@latest" %}
パッケージの作成に関する詳しい情報については[maven.apache.orgのドキュメンテーション](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)を参照してください。
{% endif %}

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <activeProfiles>
    <activeProfile>github</activeProfile>
  </activeProfiles>

  <profiles>
    <profile>
      <id>github</id>
      <repositories>
        <repository>
          <id>central</id>
          <url>https://repo1.maven.org/maven2</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
        <repository>
          <id>github</id>
          <name>GitHub OWNER Apache Maven Packages</name>
          <url>https://maven.pkg.github.com/OWNER/REPOSITORY</url>
        </repository>
      </repositories>
    </profile>
  </profiles>

  <servers>
    <server>
      <id>github</id>
      <username>USERNAME</username>
      <password>TOKEN</password>
    </server>
  </servers>
</settings>
```

{% if currentVersion != "free-pro-team@latest" %}
たとえば、以下の*OctodogApp*と*OctocatApp*は同じリポジトリに公開されます。

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <activeProfiles>
    <activeProfile>github</activeProfile>
  </activeProfiles>

  <profiles>
    <profile>
      <id>github</id>
      <repositories>
        <repository>
          <id>central</id>
          <url>https://repo1.maven.org/maven2</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
        <repository>
          <id>github</id>
          <name>GitHub OWNER Apache Maven Packages</name>
          <url>https://HOSTNAME/_registry/maven/OWNER/REPOSITORY</url>
        </repository>
      </repositories>
    </profile>
  </profiles>

  <servers>
    <server>
      <id>github</id>
      <username>USERNAME</username>
      <password>TOKEN</password>
    </server>
  </servers>
</settings>
```
{% endif %}

#### `GITHUB_TOKEN`での認証

{% data reusables.package_registry.package-registry-with-github-tokens %}

### パッケージを公開する

{% data reusables.package_registry.default-name %} たとえば、{% data variables.product.prodname_dotcom %}は`OWNER/test`というリポジトリ内の`com.example:test`という名前のパッケージを公開します。

同じリポジトリに複数のパッケージを公開したい場合には、そのリポジトリのURLを*pom.xml*ファイルの`<distributionManagement>`要素に含めてください。 {% data variables.product.prodname_dotcom %} will match the repository based on that field. リポジトリ名も`distributionManagement`要素の一部なので、複数のパッケージを同じリポジトリに公開するための追加手順はありません。

パッケージの作成に関する詳しい情報については[maven.apache.orgのドキュメンテーション](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)を参照してください。

1. パッケージディレクトリ内にある*pom.xml*ファイルの`distributionManagement`要素を編集し、`OWNER`をリポジトリを所有しているユーザもしくはOrganizationアカウントの名前で、`REPOSITORY`をプロジェクトを含むリポジトリの名前で置き換えてください。

`servers`タグの中に、子として`server`タグを`id`付きで追加し、*USERNAME*を{% data variables.product.prodname_dotcom %}のユーザ名で、*TOKEN*を個人アクセストークンで置き換えてください。
  {% if currentVersion != "free-pro-team@latest" %}
  パッケージの作成に関する詳しい情報については[maven.apache.orgのドキュメンテーション](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)を参照してください。
  {% endif %}
  ```
  <distributionManagement>
     <repository>
       <id>github</id>
       <name>GitHub OWNER Apache Maven Packages</name>
       <url>https://maven.pkg.github.com/OWNER/REPOSITORY</url>
     </repository>
  </distributionManagement>
  ```
  {% if currentVersion != "free-pro-team@latest" %}
  たとえば、以下の*OctodogApp*と*OctocatApp*は同じリポジトリに公開されます。
  ```
  <distributionManagement>
     <repository>
       <id>github</id>
       <name>GitHub OWNER Apache Maven Packages</name>
       <url>https://HOSTNAME/_registry/maven/OWNER/REPOSITORY</url>
     </repository>
  </distributionManagement>
  ```
  {% endif %}
2. パッケージを公開します。

   ```shell
   $ mvn deploy
  ```

{% data reusables.package_registry.viewing-packages %}

### パッケージをインストールする

{% data variables.product.prodname_registry %}からApache Mavenパッケージをインストールするには、*pom.xml*ファイルを編集してパッケージを依存関係として含めてください。 複数のリポジトリからパッケージをインストールしたい場合は、それぞれについて`repository`タグを追加してください。 プロジェクト内での*pom.xml*ファイルの利用に関する詳しい情報については、Apache Mavenドキュメンテーション中の「[ Introduction to the POM](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)」を参照してください。

{% data reusables.package_registry.authenticate-step %}
2. パッケージの依存関係をプロジェクトの*pom.xml*ファルの`dependencies`要素に追加し、`com.example:test`をパッケージで置き換えてください。

  ```
  <dependencies>
    <dependency>
      <groupId>com.example</groupId>
      <artifactId>test</artifactId>
      <version>1.0.0-SNAPSHOT</version>
    </dependency>
  </dependencies>
  ```
3. パッケージをインストールします。

  ```shell
  $ mvn install
  ```

### 参考リンク

- [{% data variables.product.prodname_registry %}で利用するためのGradleの設定](/packages/using-github-packages-with-your-projects-ecosystem/configuring-gradle-for-use-with-github-packages)
- [パッケージの削除](/packages/publishing-and-managing-packages/deleting-a-package/)
