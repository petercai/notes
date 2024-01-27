# Github: 使用 Github Action CI/CD Maven 自动化打包 JAR 发布

本篇文章主要是对我最近使用 **Github Action** 的一些总结，自己以前有一个需求，就是希望写完代码上传到 Github 之后自动发布 Release，为了方便以后下载以备不时只需，所以花了点时间研究了一下[自动化测试](https://so.csdn.net/so/search?q=%E8%87%AA%E5%8A%A8%E5%8C%96%E6%B5%8B%E8%AF%95&spm=1001.2101.3001.7020)和部署，发现还挺好用的,这里主要就说一下我的配置逻辑，关于 Github Action 相关的知识还需自行阅读

> [Github Action 文档](https://docs.github.com/cn/actions/quickstart)

运行流程
----

1.  监听 Master 分支 Push 事件
2.  运行 [actions/checkout@v3](https://github.com/actions/checkout) 下载代码
3.  下载 JDK [actions/setup-java@v3](https://github.com/actions/setup-java)
4.  配置 Maven [whelk-io/maven-settings-xml-action@v20](https://github.com/whelk-io/maven-settings-xml-action)
5.  获取 Maven pom 版本变量
6.  打包 [marvinpinto/action-automatic-releases@latest](https://blog.csdn.net/qq_41091006/article/details/marvinpinto/action-automatic-releases@latest)

随后每当你进行代码的 Push 操作后，Github Action 会对你上传的代码编译、发布 Release，非常的人性化.

当然我们还可以使用 Maven Deploy 部署到相关的 Maven 仓库，这里不做详细说明,可以查看[文档](https://docs.github.com/cn/actions/publishing-packages/publishing-java-packages-with-maven)进行了解.

配置文件
----

```yaml
name: Java CI with Maven


on:
  push:
    paths:
      - '**/*src/**/*.java'

jobs:
  build:

    permissions: write-all
    runs-on: ubuntu-latest
    steps:

    - uses: actions/checkout@v3
    - name: Set up JDK 11

      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
        cache: maven

    - name: maven-settings-xml-action
      uses: whelk-io/maven-settings-xml-action@v20
      with:
        repositories: ''
        servers: ''

    - name: Build with Maven
      run: mvn -B package --file pom.xml

    - run: mkdir staging && cp target/*.jar staging

    - name: Set Release version env variable
      run: |
        echo "RELEASE_VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)" >> $GITHUB_ENV
    - name: "Build & test"
      run: |
        echo "done!"

    - uses: "marvinpinto/action-automatic-releases@latest"
      with:
        repo_token: "${{ secrets.GITHUB_TOKEN }}"
        automatic_release_tag: "${{ env.RELEASE_VERSION }}"
        prerelease: false
        title: "Release ${{ env.RELEASE_VERSION }}"
        files: |
          staging/*.jar

```