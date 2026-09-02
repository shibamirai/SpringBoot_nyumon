# 手を動かして分かる Spring Boot 入門 正誤表 (2026/6/5 現在)

本では Spring Boot のバージョン 3 系が使用されていますが、ここでは最新の 4 系を使用する場合の変更点をまとめ、ソースコードとともに公開します。ソースコードは章ごとにコミットしているので、途中経過のコードも確認できます。

## 2 章 開発環境の構築

### 2-1 Eclipse のインストール

Spring Boot の 4 系では Java 25 が推奨されているため、Pleiades から最新の Eclipse をインストールしてください。

### 2-2 Spring Boot プロジェクトの作成

- プロジェクトの初期設定では Java バージョンは 25 を選んでください。
- Spring Boot バージョン 4.0.6 で動作することを確認しています。

## 6 章 画面作成

### 6-1 ライブラリの使用――webjars

- Bootstrap v5.3 系の最新バージョンは 5.3.8 (2026/6/5 現在)です
- webjars-locator ではなく、後継の webjars-locator-lite を利用します

#### リスト 6-1 pom.xml
```xml
    <!-- webjars-bootstrap -->
    <dependency>
      <groupId>org.webjars</groupId>
      <artifactId>bootstrap</artifactId>
      <version>5.3.8</version>
    </dependency>
    <!-- webjars-locator-lite -->
    <dependency>
      <groupId>org.webjars</groupId>
      <artifactId>webjars-locator-lite</artifactId>
    </dependency>

```

## 10 章 MyBatis 基本編

### 10-2 INSERT 文

MyBatis は Spring Boot 4 に対応したバージョンを使用します。ModelMapper も最新のものを使用します。

#### リスト 6-1 pom.xml
```xml
    <!-- MyBatis -->
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>4.0.1</version>
      <scope>compile</scope>
    </dependency>
    <!-- Model Mapper -->
    <dependency>
      <groupId>org.modelmapper.extensions</groupId>
      <artifactId>modelmapper-spring</artifactId>
      <version>3.2.6</version>
      <scope>compile</scope>
    </dependency>
```

※ 2026/7/24 現在、modelmapper-spring 3.2.6 を使用すると起動時に「A terminally deprecated method in sun.misc.Unsafe has been called」と Warning がでます。動作に支障はありませんが、この問題は対処されずに長らく放置されたままになっているため、使用できなくなる可能性があります。

ModelMapper の代替としては [MapStruct](https://mapstruct.org/) があるので、ポートフォリオ作成時にはそちらの使用も検討してください。  
使い方は[こちら](https://github.com/shibamirai/SpringBoot_nyumon/blob/main/MapStructの使い方.md)

## 13 章 AOP

### 13-2 AOP の実装 (1) Before・After

最後にユーザー一覧画面にアクセスしたときに出力されるコンソールのログが間違っています。

```
com.example.demo.aspect.LogAspect : メソッド開始: Page com.example.demo.user.domain.service.impl.UserServiceImpl2.getUsers(MUser,Pageable)
com.example.demo.aspect.LogAspect : メソッド終了: Page com.example.demo.user.domain.service.impl.UserServiceImpl2.getUsers(MUser,Pageable)
```

### 13-3 AOP の実装 (2) Around

最後にログイン画面にアクセスしたときに出力されるコンソールのログが間違っています。

```
com.example.demo.aspect.LogAspect : メソッド開始(Controller): String com.example.demo.login.controller.LoginController.getLogin()
com.example.demo.aspect.LogAspect : メソッド終了(Controller): String com.example.demo.login.controller.LoginController.getLogin()
```

## 14 章 例外処理

### 14-5 @AfterThrowing アスペクトでの例外処理

リスト 14-3 のファイル名が間違っています。正しくは ErrorLogAspect.java です。

## 15 章 Spring Security

### 15-3 Step 1 認証要否の設定

SecurityConfig で使用する PathRequest クラスのパッケージが誤っています。

#### リスト 15-2 SecurityConfig.java

```Java
package com.example.demo.config;

import org.springframework.boot.security.autoconfigure.web.servlet.PathRequest;  <-- 修正
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    /** セキュリティの対象外を設定 */
    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        // セキュリティ対象外の設定
        http
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers(PathRequest.toStaticResources().atCommonLocations()).permitAll()
                        .requestMatchers("/login").permitAll()
                        .requestMatchers("/user/signup").permitAll()
                        .requestMatchers("/error").permitAll()
                        .requestMatchers("/h2-console/**").permitAll()
                        .anyRequest().authenticated());
        // CSRFを無効（一時的）
        http.csrf(csrf -> csrf.disable());
        // ヘッダー設定
        http.headers(headers -> headers.frameOptions(option -> option.disable()));
        return http.build();
    }
}
```

書籍の範囲を超えますが、H2 コンソールを使用する場合のセキュリティ設定を独立した Bean にすることが可能です。H2 データベースを使用しない場合はこの Bean (h2ConsoleSecurityFilterChain()) は不要です（securityFilterChain の @Order(2) も必要ありません）。

#### SecurityConfig.java

```Java
package com.example.demo.config;

import org.springframework.boot.security.autoconfigure.web.servlet.PathRequest;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.HeadersConfigurer.FrameOptionsConfig;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    /** H2 コンソール用のセキュリティ設定 */
    @Bean
    @Order(1)
    SecurityFilterChain h2ConsoleSecurityFilterChain(HttpSecurity http) throws Exception {
        http
                // H2 コンソールのURLに対してのみこの設定を適用する
                .securityMatcher(PathRequest.toH2Console())
                // すべてのリクエストを許可
                .authorizeHttpRequests(authorize -> authorize
                        .anyRequest().permitAll())
                // H2 コンソールでは常に CSRF を無効
                .csrf(csrf -> csrf
                        .ignoringRequestMatchers(PathRequest.toH2Console()))
                // H2 コンソール用ヘッダー設定
                .headers(headers -> headers
                        .frameOptions(FrameOptionsConfig::disable));
        return http.build();
    }

    /** このアプリでのセキュリティの対象外を設定 */
    @Bean
    @Order(2)
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        // セキュリティ対象外の設定
        http
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers(PathRequest.toStaticResources().atCommonLocations()).permitAll()
                        .requestMatchers("/login").permitAll()
                        .requestMatchers("/user/signup").permitAll()
                        .requestMatchers("/error").permitAll()
                        .anyRequest().authenticated());
        // CSRFを無効（一時的）
        http.csrf(csrf -> csrf.disable());
        return http.build();
    }
}
```

### 15-9 Step 7 Remember-Me 認証

Remember-Me 作成前の動作確認で Chrome 設定を **disable** にする指示がありますが、2026/09/02 現在、このDevTools Project Settings の設定がなくなっています。このため Chrome で開発者ツールを開いたままログインすると一度エラーになります（エラー画面のHTMLが画面に表示される）。こうなってしまってもブラウザの戻るボタンで戻って再度ログインすればログインできますし、ログイン時に開発者ツールを閉じていればエラーは発生しないので、ここの Chrome 設定の変更は無視してそのまま Cookie の確認を行ってください。

## 16 章 ログ

### 16-5 ログレベルの変更

実行時に開くのはユーザー登録画面ではなくログイン画面です
