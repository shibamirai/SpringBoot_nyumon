# MapStruct の使い方

## MapStruct とは

[MapStruct](https://mapstruct.org/) は、注釈処理（アノテーションプロセッサ）を用いて、マッピングを行うコードを自動生成してくれるものです。

## ModelMapper との違い

ModelMapperは**リフレクション**という仕組みによって動的にマッピングを行うため、データが大量なときはオーバーヘッドが大きくなりますが、MapStructは直接Getter/Setterを呼び出すコードを生成するだけなので非常に高速です。

一方で、ModelMapperはわずかな設定をすればほぼ命名規則だけで動きますが、MapStructではマッピング対象ごとにインターフェースの定義とアノテーションの記述が必要になります。

## インストール方法

Eclipse で Lombok と一緒に使う場合の pom.xml の設定です。

```config
...
	<properties>
		<java.version>25</java.version>
		<m2e.apt.activation>jdt_apt</m2e.apt.activation>        <!-- (1) -->
		<org.mapstruct.version>1.6.3</org.mapstruct.version>    <!-- (2) -->
	</properties>
...
    <dependencies>
...
		<dependency>                                            <!-- (3) -->
			<groupId>org.mapstruct</groupId>
			<artifactId>mapstruct</artifactId>
			<version>${org.mapstruct.version}</version>
		</dependency>
    </dependencies>
...
	<build>
		<plugins>
...
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<executions>
					<execution>
						<id>default-compile</id>
						<phase>compile</phase>
						<goals>
							<goal>compile</goal>
						</goals>
						<configuration>
							<annotationProcessorPaths>
								<path>
									<groupId>org.projectlombok</groupId>
									<artifactId>lombok</artifactId>
								</path>
								<path>                         <!-- (4) -->
									<groupId>org.mapstruct</groupId>
									<artifactId>mapstruct-processor</artifactId>
									<version>${org.mapstruct.version}</version>
								</path>
								<path>                         <!-- (5) -->
									<groupId>org.projectlombok</groupId>
									<artifactId>lombok-mapstruct-binding</artifactId>
									<version>0.2.0</version>
								</path>
							</annotationProcessorPaths>
						</configuration>
					</execution>
					<execution>
						<id>default-testCompile</id>
						<phase>test-compile</phase>
						<goals>
							<goal>testCompile</goal>
						</goals>
						<configuration>
							<annotationProcessorPaths>
								<path>
									<groupId>org.projectlombok</groupId>
									<artifactId>lombok</artifactId>
								</path>
								<path>                         <!-- (4) -->
									<groupId>org.mapstruct</groupId>
									<artifactId>mapstruct-processor</artifactId>
									<version>${org.mapstruct.version}</version>
								</path>
								<path>                         <!-- (5) -->
									<groupId>org.projectlombok</groupId>
									<artifactId>lombok-mapstruct-binding</artifactId>
									<version>0.2.0</version>
								</path>
							</annotationProcessorPaths>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
```
(1) Eclipse にアノテーションプロセッサの設定を適用させるための設定。これがないと Eclipse では「Mapperクラスが見つからない」というエラーが出る  
(2) MapStruct のバージョンを指定  
(3) 依存関係にMapStruct を追加  
(4) コンパイル時にコードを自動作成させるための設定 **※ lombok の後に書くこと！**  
(5) Lombok と MapStruct を併用するための設定  

## 使い方

User クラスと UserForm クラスの相互マッピングを行う場合

User.java
```java
import lombok.Data;

@Data
public class User {
	private String username;
	private int age;
}
```

UserForm.java
```java
import lombok.Data;

@Data
public class UserForm {
	private String name;
	private int age;
}
```

マッパーインターフェースを作成し、マッピングメソッドを定義する。マッピングするフィールド名が異なる場合は @Mapping アノテーションでマッピングするフィールドを指定する

UserConverter.java
```java
import org.mapstruct.Mapper;

@Mapper(componentModel = "spring")		// componentModel = "spring" によって Spring のコンポーネントとして登録される
public interface UserConverter {

	/* UserForm から User へのマッピング */
	@Mapping(source = "name", target = "username")
	User fromForm(UserForm form);

	/* User から UserForm へのマッピング */
	@Mapping(source = "username", target = "name")
	UserForm toForm(User user);
}
```

自動生成されたマッピングインターフェースの実装クラスを、コントローラーなどでDIして使用する

UserController.java
```java
@Controller
@RequiredArgsConstructor
public class UserController {
	private final UserService userService;
	private final UserConverter userConverter;	// DI でマッピング実装クラスをインジェクション

	@PostMapping("/create")
	public String postCreate(@ModelAttribute @Validated UserForm form, BindingResult bindingResult) {
		...
		User user = userConverter.fromForm(form);	// UserForm から User への変換
		...
	}

	@GetMapping("{id}/edit")
	public String getEdit(Model model) {
		User user = userService.get(id);
		UserForm form = userConverter.toForm(user);	// User から UserForm への変換
		model.addAttribute("userForm", form);
		...
	}
}
```

その他の使い方は[公式のリファレンスガイド](https://mapstruct.org/documentation/reference-guide/)を参照。

実装例は[こちら](https://github.com/shibamirai/todolist-springboot4)