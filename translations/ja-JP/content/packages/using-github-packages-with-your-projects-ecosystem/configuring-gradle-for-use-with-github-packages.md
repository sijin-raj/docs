---
title: GitHub Packagesで利用するためにGradleを設定する
intro: '{{ site.data.variables.product.prodname_registry }} にパッケージを公開し、{{ site.data.variables.product.prodname_registry }} に保存されたパッケージを依存関係としてJavaプロジェクトで利用するようGradleを設定できます。'
product: '{{ site.data.reusables.gated-features.packages }}'
redirect_from:
  - /articles/configuring-gradle-for-use-with-github-package-registry
  - /github/managing-packages-with-github-package-registry/configuring-gradle-for-use-with-github-package-registry
  - /github/managing-packages-with-github-packages/configuring-gradle-for-use-with-github-packages
versions:
  free-pro-team: '*'
  enterprise-server: '>=2.22'
---

{{ site.data.reusables.package_registry.packages-ghes-release-stage }}

**ノート:** dockerイメージをインストールしたり公開したりする際に、現時点で{{ site.data.variables.product.prodname_registry }}はWindowsイメージのような外部レイヤーはサポートしていません。

### {{ site.data.variables.product.prodname_registry }} への認証を行う

{{ site.data.reusables.package_registry.authenticate-packages }}

#### 個人アクセストークンでの認証

{{ site.data.reusables.package_registry.required-scopes }}

Gradle GroovyもしくはKotlin DSLを使って、Gradleで{{ site.data.variables.product.prodname_registry }}に認証を受けることができます。それには、*build.gradle*ファイル（Gradle Groovy）もしくは*build.gradle.kts*ファイル（Kotlin DSL）ファイルを編集して、個人アクセストークンを含めます。 リポジトリ中の単一のパッケージもしくは複数パッケージを認識するようにGradle Groovy及びKotlin DSLを設定することもできます。

{% if currentVersion != "free-pro-team@latest" %}
Replace *REGISTRY-URL* with the URL for your instance's Maven registry. If your instance has subdomain isolation enabled, use `maven.HOSTNAME`. If your instance has subdomain isolation disabled, use `HOSTNAME/_registry/maven`. In either case, replace *HOSTNAME* with the host name of your {{ site.data.variables.product.prodname_ghe_server }} instance.
{% endif %}

*USERNAME*を{{ site.data.variables.product.prodname_dotcom }}のユーザ名で、*TOKEN*を個人アクセストークンで、*REPOSITORY*を公開したいパッケージを含むリポジトリの名前で、*OWNER*をリポジトリを所有する{{ site.data.variables.product.prodname_dotcom }}のユーザもしくはOrganizationアカウント名で置き換えてください。 {{ site.data.reusables.package_registry.lowercase-name-field }}

{% note %}

**Note:** {{ site.data.reusables.package_registry.apache-maven-snapshot-versions-supported }} 例として「[{{ site.data.variables.product.prodname_registry }}で使用するためのApache Mavenの設定](/packages/using-github-packages-with-your-projects-ecosystem/configuring-apache-maven-for-use-with-github-packages)」を参照してください。

{% endnote %}

##### リポジトリ中の単一のパッケージのためにGradle Groovyを使う例

```shell
plugins {
    id("maven-publish")
}

publishing {
    repositories {
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/<em>OWNER</em>/<em>REPOSITORY</em>")
            credentials {
                username = project.findProperty("gpr.user") ?: System.getenv("<em>USERNAME</em>")
                password = project.findProperty("gpr.key") ?: System.getenv("<em>TOKEN</em>")
            }
        }
    }
    publications {
        gpr(MavenPublication) {
            from(components.java)
        }
    }
}
```

##### 同じリポジトリ中の複数のパッケージのためにGradle Groovyを使う例

```shell
plugins {
    id("maven-publish") apply false
}

subprojects {
    apply plugin: "maven-publish"
    publishing {
        repositories {
            maven {
                name = "GitHubPackages"
                url = uri("https://maven.pkg.github.com/<em>OWNER</em>/<em>REPOSITORY</em>")
                credentials {
                    username = project.findProperty("gpr.user") ?: System.getenv("<em>USERNAME</em>")
                    password = project.findProperty("gpr.key") ?: System.getenv("<em>TOKEN</em>")
                }
            }
        }
        publications {
            gpr(MavenPublication) {
                from(components.java)
            }
        }
    }
}
```

##### 同じリポジトリ中の単一パッケージのためにKotlin DSLを使う例

```shell
plugins {
    `maven-publish`
}

publishing {
    repositories {
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/<em>OWNER</em>/<em>REPOSITORY</em>")
            credentials {
                username = project.findProperty("gpr.user") as String? ?: System.getenv("<em>USERNAME</em>")
                password = project.findProperty("gpr.key") as String? ?: System.getenv("<em>TOKEN</em>")
            }
        }
    }
    publications {
        register<MavenPublication>("gpr") {
            from(components["java"])
        }
    }
}
```

##### 同じリポジトリ中の複数パッケージのためにKotlin DSLを使う例

  ```shell
  plugins {
  `maven-publish` apply false
  }

  subprojects {
  apply(plugin = "maven-publish")
  configure<PublishingExtension> {
  repositories {
  maven {
  name = "GitHubPackages"
  url = uri("https://maven.pkg.github.com/<em>OWNER</em>/<em>REPOSITORY</em>")
  credentials {
  username = project.findProperty("gpr.user") as String? ?: System.getenv("<em>USERNAME</em>")
  password = project.findProperty("gpr.key") as String? ?: System.getenv("<em>TOKEN</em>")
  }
  }
  }
  publications {
  register<MavenPublication>("gpr") {
  from(components["java"])
  }
  }
  }
  }
  ```

  #### `GITHUB_TOKEN`での認証

  {{ site.data.reusables.package_registry.package-registry-with-github-tokens }}

  Mavenで `GITHUB_TOKEN` を使用する方法の詳細については、「[MavenでJavaパッケージを公開](/actions/language-and-framework-guides/publishing-java-packages-with-maven#publishing-packages-to-github-packages) 」を参照してください。

  ### パッケージを公開する

  {{ site.data.reusables.package_registry.default-name }} たとえば、{{ site.data.variables.product.prodname_dotcom }}は`OWNER/test` {{ site.data.variables.product.prodname_registry }}リポジトリ内の`com.example.test`という名前のパッケージを公開します。

  {{ site.data.reusables.package_registry.viewing-packages }}

  {{ site.data.reusables.package_registry.authenticate-step}}
  2. パッケージを作成した後、そのパッケージを公開できます。

   ```shell
   $ gradle publish
  ```

### パッケージをインストールする

プロジェクトの依存関係としてパッケージを追加することで、パッケージをインストールできます。 詳しい情報については、Gradleのドキュメンテーションの 「[ Declaring dependencies](https://docs.gradle.org/current/userguide/declaring_dependencies.html)」を参照してください。

{{ site.data.reusables.package_registry.authenticate-step}}
2. *build.gradle*ファイル（Gradle Groovy）もしくは*build.gradle.kts*ファイル（Kotlin DSL）にパッケージの依存関係を追加してください。

  Gradle Groovyの例：
  ```shell
  dependencies {
  implementation 'com.example:package'
  }
  ```
  Kotlin DSLの例：
  ```shell
  dependencies {
  implementation("com.example:package")
  }
  ```

3. *build.gradle*ファイル（Gradle Groovy）もしくは*build.gradle.kts*ファイル（Kotlin DSL）にmavenプラグインを追加してください。

  Gradle Groovyの例：
  ```shell
  plugins {
  id 'maven'
  }
  ```
  Kotlin DSLの例：
  ```shell
  plugins {
  `maven`
  }
  ```

  3. パッケージをインストールします。

  ```shell
  $ gradle install
  ```

### 参考リンク

- [{{ site.data.variables.product.prodname_registry }}で利用するためのApache Mavenの設定](/packages/using-github-packages-with-your-projects-ecosystem/configuring-apache-maven-for-use-with-github-packages)
- [パッケージの削除](/packages/publishing-and-managing-packages/deleting-a-package/)