# √ 安全 <Badge text="进行中" type="warning"/>

即使只限定在“软件架构设计”这个语境下，系统安全仍然是一个很大的话题。我们谈论的计算机系统安全，远不仅指“防御系统被黑客攻击”这样狭隘的“安全”。架构安全性至少应包括了（不限于）以下这些问题的具体解决方案：

- **认证**（Authentication）：系统如何正确分辨出操作用户的真实身份？

- **授权**（ Authorization）：系统如何控制一个用户该看到哪些数据、能操作哪些功能？

- **凭证**（Credentials）：系统如何保证它与用户之间的承诺是双方当时真实意图的体现，是准确、完整且不可抵赖的？

- **保密**（Confidentiality）：系统如何保证敏感数据无法被包括系统管理员在内的内外部人员所窃取、滥用？

- **验证**（Verification）：系统如何确保提交到每项服务中的数据是合乎规则的，不会对系统稳定性、数据一致性、正确性产生风险？

- **传输安全**（Transport Security）：系统如何保证通过网络传输的信息无法被第三方窃听、篡改和冒充？

- **漏洞利用**（Exploit）：系统如何避免在基础设施和应用程序中出现弱点，被攻击者利用？

- ……

上面这些安全相关的问题，解决起来确实是既繁琐复杂，又难以或缺。值得庆幸的是这一部分内容基本上都是与具体系统、具体业务无关的通用性问题、这意味着它们会存在着业界通行的，已被验证过是行之有效的解决方案，乃至已经形成某一些行业标准，不需要我们自己从头去构思如何解决。后面我们将会通过标准的方案，逐一探讨以上问题的主流处理方法。

还有其他一些安全相关的内容，主要由管理、运维、审计方面负责，尽管软件架构也需要配合参与，但不列入本文的讨论范围之中，譬如：安全审计、系统备份与恢复、防治病毒、信息系统安全法规与制度、计算机防病毒制度、保护私有信息规则，等等。

## 认证

::: tip 认证（Authentication）

系统如何正确分辨出操作用户的真实身份？

:::

“认证”可以说是一个系统中最基础的安全设计，再简陋的系统大概也不大可能省略掉“用户登录”功能。但“认证”这件事情又并不如大多数人所认为的那样，校验一下 用户名、密码是否正确这么简单。尤其是在基于Java的软件系统里，尝试去触接了解Java安全标准的人往往会对一些今天看起来很别扭的概念产生疑惑。在这一部分，将简要概览一下关于认证的主流行业规范、标准；项目中具体如何认证、授权的内容放到下一节去介绍。

最初的Java系统里，安全中的“认证”其实是特指“代码级安全”（你是否信任要在你的电脑中运行的代码），这是由“Java 2”之前它的主要应用形式Applets所决定的：从远端下载一段Java代码，以Applet的形式在用户的浏览器中运行，当然要保证这些代码不会损害用户的计算机才行。这一阶段的安全催生了今天仍然存在于Java体系中的“安全管理器”（java.lang.SecurityManager）、“代码权限许可”（java.lang.RuntimePermission）这些概念。

不久之后，Java迎来了互联网的迅速兴起，进入了Java第一次快速发展时期，基于超文本的Web应用迅速盖过了“Java 2”时代之前的Applet，此时“安全认证”的重点逐渐转为“用户级安全”（你是否信任正在操作的用户）。在1999年随着J2EE 1.2（它是J2EE的首个版本，版本号直接就是1.2）所发布的Servlet 2.2中增加了一系列认证的API，诸如：

- HttpServletRequest.isUserInRole()
- HttpServletRequest.getUserPrincipal()
- 还内置支持了四种硬编码、不可扩展的认证机制：BASIC、FORM、CLIENT-CERT和DIGEST。

到Java 1.3时代中，Sun公司提出了同时面向与代码级安全和用户级安全的认证授权服务JAAS（Java Authentication and Authorization Service，1.3处于扩展包中，1.4纳入标准包），不过相对而言，在JAAS中代码级安全仍然是占更主要的地位。

由于用户数据可能来自于各种不同的数据源（譬如RDBMS、JNDI、LDAP等等），JAAS设计了一种插入式（Pluggable）的认证和授权模型，以适配各种环境。在今天仍然活跃的主流安全框架中的许多概念，譬如用户叫做“Subject / Principal”、密码存在“Credentials”之中、登陆后从安全上下文“Context”中获取状态等都可以追溯到这一时期所设计的API：

- LoginModule （javax.security.auth.spi.LoginModule）
- LoginContext （javax.security.auth.login.LoginContext）
- Subject （javax.security.auth.Subject）
- Principal （java.security.Principal）
- Credentials（javax.security.auth.Destroyable、javax.security.auth.Refreshable）

但是，尽管JAAS开创了许多沿用至今的安全概念，实质上并没有得到广泛的应用。这里有两大原因，一方面是由于JAAS同时面向代码级和用户级的安全机制，使得它过度复杂化，难以推广。在这里问题上JCP一直在做着持续的增强和补救，譬如Java EE 6中的JASPIC、Java EE 8中的EE Security：

- JSR 115：[Java Authorization Contract for Containers](https://jcp.org/aboutJava/communityprocess/mrel/jsr115/index3.html)（JACC）
- JSR 196：[Java Authentication Service Provider Interface for Containers](https://jcp.org/aboutJava/communityprocess/mrel/jsr196/index2.html)（JASPIC）
- JSR 375： [Java EE Security API](https://jcp.org/en/jsr/detail?id=375)（EE Security）

而另一方面，可能是更重要的一个原因是在21世纪的第一个十年里，以EJB为代表的容器化J2EE与以“Without EJB”为口号、以Spring、Hibernate等为代表的轻量化企业级开发框架之争，以后者的胜利而结束。这也使得依赖于容器安全的JAAS无法得到大多数人的认可。

在今时今日，实际活跃于Java届的两大私有的（私有的意思是不由JSR所规范的，即没有java/javax.*作为包名的）的安全框架：

- [Apache Shiro](https://shiro.apache.org/)
- [Spring Security](https://spring.io/projects/spring-security)

相较而言，Shiro使用更为便捷易用，而Spring Security的功能则要复杂强大一些。在我们的项目中（无论是单体架构还是微服务架构），均选择了Spring Security作为安全框架。当然，这里面也有很大一部分是因为Spring Boot/Cloud全家桶的原因。这两大安全框架都解决的问题都很类似，大致可以分为四类：

- 认证：以HTTP协议中定义的各种认证、表单等认证方式确认用户身份，这是本节的主要话题。
- 授权：主要是授权结果，即访问控制（Access Control），稍后讲的“授权”将聚焦在授权的过程，尤其是多方授权中。这部分内容会放到下一节一起讨论。
- 密码的存储：就是字面意思，我们会放到“保密”这节去一起讨论。
- 安全上下文：用户获得认证之后，需要有API可以得知该用户的基本资料、用户拥有的权限、角色等。

介绍了一大段关于Java中安全标准的历史，我们最终还是要切入到如何处理认证的话题上，这可是随着网络出现就有的一个东西，所以，IETF的最初想法是基于Web的验证就应该在HTTP协议层面来解决。

> **互联网工程任务组**（Internet Engineering Task Force，IETF）：管理和发布互联网标准的组织，其标准以RFC即"请求意见稿"Request for Comments的形式发出。不仅是HTTP，几乎目前所有的主要网络协议，如IP、TCP、UDP、FTP、CMIP、SOCKS，等等都是以RFC形式定义的。

IETF给HTTP 1.1协议定义了401（Unauthorized，未授权）状态码，当服务端向客户端返回此状态码时，应在Header中附带一个WWW-Authenticate项，此项目通过跟随的一个可扩展的Scheme，告诉客户端应该采取怎样的方式来开始验证，例如：

```http
HTTP/1.1 401 Unauthorized
Date: Mon, 24 Feb 2020 16:50:53 GMT
WWW-Authenticate: Basic realm="From icyfenix.cn"
```

同时，IETF也定义了几种标准的Schema，对应了一些预定义好的认证方式，包括：

- **Basic**：[RFC 7617](https://tools.ietf.org/html/rfc7617)，HTTP基础认证，弹出一个输入框，把用户名和密码Base64之后发送出去
- **Digest**：[RFC 7616](https://tools.ietf.org/html/rfc7616)，HTTP摘要认证，弹出一个输入框，把用户名和密码加盐后再通过MD5/SHA等哈希算法摘要后发送出去
- **Bearer**：[RFC 6750](https://tools.ietf.org/html/rfc6750)，OAuth 2.0令牌（OAuth2是一个授权协议，但同时也涉及到认证的内容，下一节的主角）
- **HOBA**：[RFC 7486](https://tools.ietf.org/html/rfc7486) ，**H**TTP **O**rigin-**B**ound **A**uthentication的缩写，一种基于数字签名的认证。

因为Scheme是允许自定义扩展的，很多厂商也加入了自己的认证方式，譬如：

- **AWS4-HMAC-SHA256**：简单粗暴的名字，一看就是亚马逊AWS基于HMAC-SHA256哈希算法的认证
- **NTLM** / **Negotiate**：微软公司NT LAN Manager（NTLM）用到的两种认证方式
- **Windows Live ID**：这个不需要解释了
- **Twitter Basic**：一个不存在的网站所改良的HTTP基础认证
- ……

现在主流的信息系统，直接采用上面这些认证方式比例不算太高，目前的主流仍是Form表单认证，即我们通常所说的“登陆页面”。表单认证并没有什么行业标准可循，表单中的用户字段、密码字段、验证码字段、是否要在客户端加密、加密的方式、接受表单的服务入口等都可由服务端、客户端自行协商决定。

在Fenix's Bookstore项目中，我们所设计的登录实质上也是一种表单认证，借用了Spring Security的认证管理器。Spring Security中提供了默认的登陆表单界面和配套的服务，只要在Spring Security的Web安全中简单配置即可启用：

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/").permitAll() // 首页地址'/'的请求都允许访问
                .anyRequest().authenticated() // 任何请求,登录后才可以访问
                .and()
                .formLogin()  // 启用表单登录认证，还有另一种httpBasic()方法代表了HTTP基础认证
                .permitAll(); // 登录页面用户任意访问
                .and()
                .logout().permitAll(); // 注销的服务任意访问
    }    
}
```

Spring Security的权限控制措施在两个层面进行，一种Web级别的访问控制，这是在Web服务器中附加的过滤器（FilterSecurityInterceptor）实现的，另一种是方法级权限控制，是通过动态代理实现的。第二种将在下一节“授权”部分中提及，这里先来说第一种。

当Spring Security被启动时（在Spring Boot中通过@EnableWebSecurity注解启动），将会在Web服务器中附加十几个不同作用的过滤器，譬如上面代码就直接涉及到其中三个：

- SecurityContextPersistenceFilter：用于维护安全上下文，“上下文”说白了就是如果用户登陆了系统，那服务的代码中总该有个地放可以取到当前登陆用户是谁这类信息
- UsernamePasswordAuthenticationFilter：用于完成用户名、密码的验证过程
- LogoutFilter：用于注销
- FilterSecurityInterceptor：用于Web级别的访问控制，如果设置了指定地址需要登陆而实际未登陆，或者设定了需要某些权限才能访问而实际用户并没有，那将抛出AuthenticationException与AccessDeniedException异常

让我们再回到上面的代码，这段简单的工作流程是：

1. 启用过滤器UsernamePasswordAuthenticationFilter，在其attemptAuthentication()方法中，会从Request中获取用户名和密码，传递给认证管理器AuthenticationManager的authenticate()方法
3. 认证管理器的目的是协调不同的用户来源，譬如来自数据库、来自LDAP、来自OAuth等等，每一个用户来源都应该有一个实现了AuthenticationProvider接口并注册到认证管理器的实现类所代表，认证管理器将根据需要，调用对应Provider的authenticate()方法实际完成认证操作。
4. Spring Security默认的Provider是DaoAuthenticationProvider，它在Bookstore项目中并未被采用，而是另外实现了一个UsernamePasswordAuthenticationProvider。但是两者的实际逻辑是相似的，都是调用UserDetailsService接口里的loadUserByUsername()来获取用户信息，UserDetailsService是读取用户明细数据的接口，Spring Security并不关心用户系统的实际存储结构，但认证时肯定也必须使用到用户信息，默认使用InMemoryUserDetailsManager，也就是从内存中写死一些用户数据来完成。
6. 在AuthenticationProvider中比较传入的用户密码与数据库中的用户密码是否一致（具体怎么个比较法将在“保密”这一节中说明），返回结果，完成认证。

以上流程是大多数系统，尤其是单体系统中主流的认证方式，哪怕不基于Apache Shiro或Spring Security来实现，其思路很可能也是与上面描述的差不多的。但我们的Bookstore却并未直接应用这种认证方式，而是借用了OAuth2授权协议中的密码授权模式，在此过程中完成认证。为何会选择这种方式，以及具体实现部分的内容，将在下一部分“授权”中继续介绍。

## 授权

::: tip 授权（ Authorization）

系统如何控制一个用户该看到哪些数据、能操作哪些功能？

:::

“授权”这个行为通常伴随着“认证“、”账号“共同出现，并称为AAA（Authentication、Authorization、Account）。授权行为在程序中其实非常普遍，我们给一个类、一个方法设置范围控制符（public、protected、private、\<Package\>），这其实也是一种授权（访问控制）行为。授权涉及到了两个相对独立的问题：

- 确保授权的过程可靠：对于单一系统来说，授权的过程是比较容易做到可控的，以前很多语境上提到授权，实质上讲的都是访问控制，理论上两者是应该分开的。而在涉及多方的系统中，授权过程就是一个必须严肃对待的问题：如何即让第三方系统能够访问到所需的资源，又能保证其不泄露用户的敏感数据？现在常用的多方授权协议主要有OAuth2和SAML 2.0（注意这两个协议涵盖的功能并不是直接对等的）。

- 确保授权的结果可控：授权的结果往往是用于对程序功能或者资源的访问控制（Access Control），形成理论的权限控制模型有：自主访问控制（Discretionary Access Control，DAC）、强制访问控制（Mandatory Access Control，MAC）、基于属性的权限验证（Attribute-Based Access Control，ABAC）还有最为常用，也相对通用的是基于角色的权限模型（Role-Based Access Control，RBAC）。

由于篇幅原因，在这个小节里我们只介绍（将要）使用到的，也是最常用到的RBAC和OAuth2。先来说较为简单的RBAC。

### RBAC

所有的访问控制模型，实质上都是在解决同一个问题：“**谁**（User）”拥有什么"**权限**（Authority）"去操作哪些“**资源**（Resource）”

这个问题看起来并不难，最直观的解决方案就是在用户对象上，设定一些操作权限，在使用资源时，检查是否有对应的操作权限即可。是的，请不要因太过简单直接而产生疑惑——Spring Security的访问控制本质上就是这么做的。不过，这种把操作权限直接关联在用户身上的简单设计，在复杂系统上确实会导致比较繁琐的操作。试想一下，如果某个系统涉及到成百上千的资源，又有成千上万的用户，要为每个用户分配合适的权限将带来务必庞大的操作量和极高的出错概率，这也即是RBAC所要解决的问题。

为了避免对每一个用户设定权限，RBAC将权限从用户身上剥离，改为绑定到“**角色**（Role）”上，一种我们常见的RBAC应用就是操作系统权限中的“用户组”，这就是一种角色。用户可以隶属与一个或者多个角色，某个角色中也会包含有多个用户，角色之间还可以有继承性（父、子角色的权限继承，RBAC1）。这样，资源的操作就只需按照有限且相对固定的角色去分配操作权限，而不去面对随时会动态增加的用户去分配。当用户的职责发生变化时，在系统中就体现为改变他所隶属的角色，譬如将“普通用户角色”改变“管理员角色”，就可以迅速完成其权限的调整，降低了权限分配错误的风险。RBAC的主要元素之间的关系可以以下图来表示：
<mermaid>
graph LR
    User("用户（User）") --隶属--> Role("角色（Role）") 
    Role --拥有-->Permission("许可（Permission）") 
</mermaid>
上图中出现了一个新的名词“**许可**（Permission）”。所谓的许可，就是抽象权限的具象化体现。权限在系统中的含义应该是“允许何种**操作**作用于哪些**数据**之上”，这个即为“许可”。举个具体的例子，譬如某个文章管理系统的UserrStory中，与访问控制相关的Backlog可能会是这样描述的：

> **Backlog**：
>
> **小周**（User）是某SCI杂志的**审稿人**（Role），职责之一是在系统中**审核论文**（Authority）。在**审稿过程**（Session）中，当他认为某篇**论文**（Resource）达到了可以公开发表标准时，就会在后台**点击通过按钮**（Operation）来完成审核。

以上，“给论文点击通过按钮”就是一种许可（Permission），它是“审核论文”这项权限（Authority）的具象化体现。

与微服务架构中的完全遵循RBAC进行访问控制的Kubernetes不同，我们在单体架构中使用的Spring Security参考了但并没有完全按照RBAC来进行设计。Spring Security的设计里用户和角色都可以拥有权限，譬如在HttpSecurity对象上，就同时有着hasRole()和hasAuthority()方法，可能有不少刚接触的人会疑惑，混淆它们之间的关系。在Spring Security的访问控制模型可以认为是下图所示这样的：
<mermaid>
graph LR
    User("用户（User）") --隶属--> Role("角色（Role）") 
    User --赋予--> Authority("权限（Authority）")
    Role --拥有-->Permission("许可（Permission）") 
    Authority --拥有--> Permission
</mermaid>
站在代码实现的角度来看，Spring Security中Role和Authority的差异很小，它们共同存储在同一位置，唯一的差别仅是Role会在存储时自动带上“ROLE_”前缀（可以配置的）罢了。

但在使用者的角度来看，Role和Authority的差异可以很大，你可以执行决定你的系统中到底Permission只能对应到角色身上，还是可以让用户也拥有某些角色中没有的权限。这个观点，在Spring Security自己的文档上说的很清楚：这取决于你自己如何使用。

> **The core difference between these two is the semantics we attach to how we use the feature.** For the framework, the difference is minimal – and it basically deals with these in exactly the same way.

使用RBAC，你可以控制最终用户在广义和精细级别上可以做什么。您可以指定用户是管理员，专家用户还是普通用户，并使角色和访问权限与组织中员工的身份职位保持一致。仅根据需要为员工完成工作的足够访问权限来分配权限。

### OAuth2

简要介绍过RBAC，下面我们再来看看相对要复杂繁琐写的OAuth2授权协议（顺带说一下，OAuth1.0已经完全废弃了）。先明确一件事情，OAuth2是一个多方系统中的授权协议，如果你的系统并不涉及到第三方（譬如我们单体架构的Bookstore，即不为第三方提供服务，也不使用第三方的服务），引入OAuth2其实并无必要。我们之所以把OAuth2提前引入，主要是为了给微服务架构做铺垫。

OAuth2是在[RFC 6749](https://tools.ietf.org/html/rfc6749)中定义授权协议，在RFC 6749正文的第一句就明确了OAuth2是解决第三方应用（Third-Party  Application）的授权协议。前面也说到，如果只是单方系统，授权过程是比较容易解决的，至于多方系统授权过程会有什么问题，这里举个现实的例子来说明。

譬如你现在正在阅读的这个网站（[http://icyfenix.cn](http://icyfenix.cn)），它的建设和更新大致流程是：笔者以Markdown形式写好了某篇文章，上传到由[GitHub](https://github.com)提供的[代码仓库](https://github.com/fenixsoft/awesome-fenix)，接着由[Travis-CI](https://travis-ci.com)提供的持续集成服务会检测到该仓库发生了变化，触发一次Vuepress编译活动，生成目录和静态的HTML页面，然后推送回GitHub Pages，再触发腾讯云CDN的缓存刷新。这个过程要能顺利进行，就存在一些必须解决的授权问题，Travis-CI只有得到了我的明确授权，GitHub才能同意它读取我代码仓库中的内容，问题是它该如何获得我的授权呢？一种简单粗暴的方案是我把我的用户账号和密码都告诉Travis-CI，但这显然导致了以下这些问题：

- **密码泄漏**：如果Travis-CI被黑客攻破，将导致我GitHub的密码也同时被泄漏
- **访问范围**：Travis-CI将有能力读取、修改、删除、更新我放在GitHub上的所有代码仓库
- **授权回收**：我只有修改密码才能回收授予给Travis-CI的权力，可是我在GitHub的密码只有一个，修改了意味着所有别的第三方的应用程序会全部失效

以上出现的这些问题，也就是OAuth2所要解决的问题，尤其是没有TLS支持传输安全的环境下（典型如HTTP）依然可以解决这些问题。OAuth2提出的解决办法是通过一个令牌（Token）代替用户密码作为授权的凭证，有了令牌之后，哪怕令牌被泄漏，也不会导致密码的泄漏，令牌上可以设定访问资源的范围以及时效性，每个应用都持有独立的令牌，哪个失效都不会波及其他，一下子上面提出的三个问题都解决了，有了一层令牌之后，整个授权的流程如下图所示：
<mermaid>
sequenceDiagram
    第三方应用->>+资源所有者: 要求用户给予授权
    资源所有者-->>-第三方应用: 同意给予该应用授权
    第三方应用->>+授权服务器: 我有用户授权，申请访问令牌
    授权服务器-->>-第三方应用: 同意发放访问令牌
    第三方应用->>+资源服务器: 我有访问令牌，申请开放资源
    资源服务器-->>-第三方应用: 同意开放资源
</mermaid>

这个时序图里面涉及到了OAuth2中几个关键术语，我们通过前面那个具体的上下文语境来解释其含义，这对理解后续几种认证流程十分重要：

- **第三方应用**（Third-Party  Application）：需要得到授权访问我资源的那个应用，即“Travis-CI”
- **授权服务器**（Authorization Server）：能够根据我的意愿提供授权（授权之前肯定已经进行了必要的认证过程，但这在技术上与授权可以没有直接关系）的服务，即“GitHub”
- **资源服务器**（Resource Server）：能够提供第三方应用所需资源的服务（它与认证服务可以是相同的服务器，也可以是不同的服务器），即“代码仓库”
- **资源所有者**（Resource Owner）： 拥有授权权限的人，这里即是“我”
- **操作代理**（User Agent）：指用户用来访问服务器的工具，对于指代人类的“用户”来说这个通常就是浏览器，但在微服务中一个服务经常会作为另一个服务的"用户"，此时指的可能就是HttpClient、RPCClient或者其他访问途径。

看来“用令牌代替密码”确实是解决问题的好方法，但这最多只能算个思路，距离执行步骤还是不够具体的，时序图中的“要求/同意授权”、“要求/同意发放令牌”、“要求/同意开放资源”几个服务请求、响应该如何设计，这就是执行步骤的关键了。对此，OAuth2一共提出了四种不同的授权方式（这就是我说OAuth2复杂繁琐的原因，摊手），分别为：

- 授权码模式（Authorization Code）
- 简化模式（Implicit）
- 密码模式（Resource Owner Password Credentials）
- 客户端模式（Client Credentials）

#### 授权码模式

授权码模式是四种模式中最严谨（繁琐）的，它考虑到了几乎所有敏感信息泄漏的预防和后果。具体步骤的时序如下：
<mermaid>
sequenceDiagram
	资源所有者 ->> 操作代理: 通过操作代理访问应用
	操作代理 ->> 第三方应用: 遇到需要使用的资源
	第三方应用 ->> 授权服务器: 转向授权服务器的授权页面
	资源所有者 ->>+ 授权服务器: 认证身份，同意授权
	授权服务器 -->>- 操作代理: 返回第三方应用的回调地址，附带授权码
	操作代理 ->> 第三方应用: 转向回调地址
	第三方应用 ->>+ 授权服务器: 将授权码发回给授权服务器，换取访问令牌
	授权服务器 -->>- 第三方应用: 给予访问令牌
	opt  资源访问过程 
		第三方应用 ->>+ 资源服务器: 提供访问令牌
		资源服务器 -->>- 第三方应用: 提供返回资源
		第三方应用 -->> 资源所有者: 返回对资源的处理给用户
	end
</mermaid>

在开始完成整个授权过程以前，第三方应用先要到授权服务器上进行注册，所谓注册，是指向认证服务器提供一个域名地址，从授权服务器中获取ClientID和ClientSecret，然后便可以开始如下授权过程：
1. 第三方应用将资源所有者（用户）导向授权服务器的授权页面，并向授权服务器提供ClientID及同意授权后的回调URI，这是一次客户端页面转向。
2. 授权服务器根据ClientID确认第三方应用的身份，用户在授权服务器中决定是否同意向该身份的应用进行授权（认证的过程在此之前应该已经完成）。
3. 如果用户同意授权，授权服务器将转向地第三方应用在第1步调用中提供的回调地址URI，并附带上一个授权码和获取令牌的地址作为参数，这也是一次客户端页面转向。
4. 第三方应用通过回调地址收到授权码，然后将授权码与自己的ClientSecret一起作为参数，**通过服务端**向授权服务器提供的获取令牌的服务地址发起请求，换取令牌。该服务端应与注册时提供的域名一直。
5. 授权服务器核对授权码和ClientSecret，确认无误后，向第三方应用授予令牌。令牌可以是一个或者两个，其中必定要有的是访问令牌（Access Token），可选的是刷新令牌（Refresh Token）。访问令牌用于到资源服务器获取资源，有效期较短，刷新令牌用于在访问令牌失效后重新获取，有效期较长。
6. 资源服务器根据访问令牌所允许的权限，向第三方应用提供资源。

这个过程设计，已经考虑到了几乎所有合理的意外情况，举例几个容易想到的：

- 会不会有其他应用冒充第三方应用骗取授权？<br/>
  ClientID代表一个第三方应用的“用户名”，这个是可以完全公开的。但ClientSecret应当只有应用自己才知道，这个代表了第三方应用的“密码”。在第5步发放令牌时，调用者必须能够提供ClientSecret才能成功完成。只要第三方应用妥善保管好ClientSecret，就没有人能够冒充它。
- 为什么要先发放授权码，再用授权码换令牌？<br/>
  这是因为客户端转向（通常就是一次HTTP 302重定向）对于用户是可见的，换而言之，授权码完全可能会暴露给用户（以及用户机器上的其他程序），但由于用户并没有ClientSecret，光有授权码也是无法换取到令牌的，所以避免了令牌在传输转向过程中泄漏的风险。
- 为什么要设计一个时限较长的刷新令牌和时限较短的访问令牌？不能直接把访问令牌的时间调长吗？<br/>
  这是为了缓解OAuth2在**实际应用**中的一个主要缺陷，通常访问令牌一旦发放，除非超过了令牌中的有效期，否则很难（需要付出较大代价）有其他方式让它失效，所以访问令牌的时效性一般设计的比较短（譬如几个小时），如果还需要继续用，那就定期用刷新令牌去更新，授权服务器就可以在更新过程中决定是否还要继续给予授权。至于为什么说很难让它失效，我们将放到下一节“凭证”中解释这一点。

尽管授权码模式是严谨的，但是它并不够好用，这不仅仅体现在它那繁复的调用过程上，还体现在它对第三方应用提出了一个具体的要求：必须有服务端（因为第4步要发起服务端转向，而且服务端的地址必须与注册时提供的回调URI在同一个域内）。不要觉得要求一个系统要有服务端是天经地义理所当然的事情，本站的示例程序（[http://bookstore.icyfenix.cn](http://bookstore.icyfenix.cn)）就没有服务端支持，里面使用到了GitHub Issue作为留言板，对GitHub来说照样是第三方应用，需要OAuth2授权来解决。除浏览器外，现在越来越普遍的是移动或桌面端的Client-Side Web Applications，譬如现在大量的基于Cordova、Electron、Node-Webkit.js的应用。所以在此需求里，引出了OAuth2的第二种授权模式：隐式授权。

#### 隐式授权

隐式授权省略掉了通过授权码换取令牌的步骤，整个授权过程都不需要服务端支持，一步到位。其代价是在隐式授权中，授权服务器不会再去验证第三方应用的身份（因为没有服务器了，ClientSecret没有人保管，就没有意义了。但其实还是会限制第三方应用的回调URI地址必须与注册时提供的域名一致，有可能被DNS污染之类的攻击所攻破，但仍算是尽人事努力一下）；也不能避免令牌暴露给资源所有者（以及用户机器上可能意图不轨的其他程序、HTTP的中间人攻击等）了。隐私授权的调用时序如下图（从此之后的授权模式，时序中我就不画资源访问部分的内容了，就是前面opt框中的那一部分，以便更聚焦重点）所示：
<mermaid>
sequenceDiagram
	资源所有者 ->> 操作代理: 通过操作代理访问应用
	操作代理 ->> 第三方应用: 遇到需要使用的资源
	第三方应用 ->> 授权服务器: 转向授权服务器的授权页面
	资源所有者 ->> 授权服务器: 认证身份，同意授权
	授权服务器 -->> 操作代理: 返回第三方应用的回调地址，通过Fragment附带访问令牌
	操作代理 ->> 第三方应用: 转向回调地址，通过脚本提取出Fragment中的令牌
</mermaid>

在以上过程设计中，与授权码模式模式的显著区别是授权服务器在得到用户授权后，直接返回了访问令牌，这显然降低了安全性，但OAuth2仍然努力尽可能地做到相对安全，譬如在前面提到的隐私授权中，尽管不需要用到服务端，但仍然需要在注册时提供回调域名，此时会要求该域名与接受令牌的域名处于同一个域内。此外，在隐私模式中明确禁止发放刷新令牌。

还有一点，在RFC 6749对隐式授权的描述中，特别强调了令牌是“通过Fragment带回”的。部分对超文本协议没有了解的读者，可能不知道[Fragment](https://en.wikipedia.org/wiki/Fragment_identifier)是个什么东西？

> **额外知识**
>
> In computer [hypertext](https://en.wikipedia.org/wiki/Hypertext), a **fragment identifier** is a [string](https://en.wikipedia.org/wiki/Character_string_(computer_science)) of [characters](https://en.wikipedia.org/wiki/Character_(computing)) that refers to a [resource](https://en.wikipedia.org/wiki/Resource_(computer_science)) that is subordinate to another, primary resource. The primary resource is identified by a [Uniform Resource Identifier](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) (URI), and the fragment identifier points to the subordinate resource.

不想看英文，或者看了觉得概念不好的话，我简单告诉你，Fragment就是地址中"#"号后面的部分，譬如这个地址：

> http://bookstore.icyfenix.cn/#/detail/1

后面的“/detail/1”便是Fragment，这个语法是在[RFC 3986](https://tools.ietf.org/html/rfc3986)中定义的标准，规范中解释了这是用于客户端定位的URI从属资源，譬如HTML中就可以使用Fragment来做文档内的跳转（你现在可以点击一下这篇文章左边菜单中的几个子标题，看看浏览器地址的变化）而不会发起服务端请求。此外，如果浏览器对一个带有Fragment的地址发出Ajax请求，那Fragment是不会跟随请求被发送到服务端的，只能在客户端通过Script脚本来读取。所以隐式授权巧妙地利用这个特性，尽最大努力地避免了令牌从操作代理到第三方服务之间的链路存在HTTP中间人攻击，被泄漏出去。而认证服务器到到操作代理之间的这一段链路的安全，则可以通过TLS（即HTTPS）来保证没有中间人攻击，我们可以要求认证服务器都是基于HTTPS的，但无法要求第三方应用都是基于HTTPS。

#### 密码模式

前面所说的授权码模式和隐私模式，是纯粹的授权模式，它与认证没有直接的关系，如何认证用户的真实身份这是与进行授权互相独立的过程。但在密码模式里，认证和授权就被整合成了同一个过程。

这一种模式原本是只提供给用户对第三方应用是高度可信任的场景之中，譬如第三方应用是操作系统，本应该是不太多见的。但是近年来微服务风潮兴起，反而涌现出了密码模式的一种常见应用形式，譬如微服务群中有一些应用服务与授权服务都是由同一个服务商所搭建的，这自然就可以信任它们了。在单体服务的Fenix's Bookstore实现里，就直接采用了密码模式将认证和授权统一起来，我并不需要担心通过前端代码输入用户名、密码时，前端代码会对这些敏感信息做出什么不轨的行为，因为前端代码虽然在OAuth2中相当于第三方应用的角色，但它也是我本人所提供的，所以不存在信任问题（同时再次说明，如果不是出于方便与其他架构对比的目的，那也不存在引入OAuth2把它当作第三方看待的必要）。密码模式的调用时序就很简单了，如下图所示：
<mermaid>
sequenceDiagram
	资源所有者 ->> 第三方应用: 提供密码凭证
	第三方应用 ->> 授权服务器: 发送用户的密码凭证
	授权服务器 -->> 第三方应用: 发放访问令牌和刷新令牌
</mermaid>

显然，在这种模式下，“如何保障安全”的职责无法由OAuth2的过程设计来承担，应是由用户和第三方应用来自行保障了，尽管OAuth2在规范中强调到“此模式下，第三方应用不得保存用户的密码”，但这并没有任何的约束力。

#### 客户端模式

客户端模式是四种模式中最简单的，它只涉及到两个主体，第三方应用和授权服务器。严谨一点说，现在叫第三方应用已经不合适的，因为这里已经没有了“第二方”的存在，资源所有者、操作代理都是不存在的。甚至于叫“授权”都不太恰当，资源所有者都没有了，自然也不会有谁授予谁权限的过程。

客户端模式是指应用（就不写第三方了）以自己的名义，向授权服务器申请资源许可。这通常用在一些管理或者自动处理形场景之中。举个例子，譬如我开了一家网上书店，因为小本经营，不像京东那样全国多个仓库可以调货，我得保证只要客户成功购买，我就必须有货可发，不能超卖。但经常有人下了订单又拖着不付款，导致部分货物处于冻结状态。所以我写了一个订单清理的定时服务，自动清理掉超过2分钟的未付款的订单。这件UserStory里，订单肯定属于用户自己的资源，如果把订单清理服务看作一个独立的第三方应用的话，他就不应该向用户去申请授权，而应该直接以自己的名义向授权服务器申请一个能清理所有用户订单的授权。客户端模式的时序如下图所示：
<mermaid>
sequenceDiagram
	应用 ->>+ 授权服务器: 申请授权
	授权服务器 -->>- 应用: 发放访问令牌
</mermaid>

还有一种与客户端模式类似的授权模式，在[RFC 8628](https://tools.ietf.org/html/rfc8628#section-3.4)中定义为“设备码模式（Device Code）”，这里顺便简单提一下。设备码模式用于在无输入的情况下区分设备是否允许，典型的应用便是手机锁网解锁（锁网在国内较少，但在国外很常见）或者激活（譬如某游戏机注册到某个游戏平台）的过程。时序如下图所示：
<mermaid>
sequenceDiagram
	设备 ->> 授权服务器: 要求授权
	授权服务器 -->> 设备: 提供验证URI和用户码
	loop 循环直至超时
		设备 ->> 授权服务器: 申请令牌
	end
</mermaid>

进行验证时，设备需要从授权服务器获取一个URI地址和一个用户码，然后需要用户手动或设备自动地到验证URI中输入用户码。在这个过程中，设备会一直循环，尝试去获取令牌，直到拿到令牌或者用户码过期为止。

## 凭证

::: tip 凭证（Credentials）

系统如何保证它与用户之间的承诺是双方当时真实意图的体现，是准确、完整且不可抵赖的？

:::
在前面介绍OAuth2的内容中，每一种授权模式的目的都是拿到访问令牌，但从未涉及过拿回来的令牌应该长什么样子？反而还挖了一些坑（为何说OAuth2的一个主要缺陷是令牌难以主动失效）还没有填。这节我们讨论凭证，此话题中令牌必须得是主角了，此外，我们还要在这节讨论不使用OAuth2、最传统的方式是如何完成前面所讨论的认证、授权的。

### Cookie-Session

我们知道，HTTP协议是一种无状态的传输协议，无状态是指协议对事务处理没有上下文的记忆能力，每一个请求都是完全独立的，但是我们中肯定有许多人并没有意识到HTTP协议无状态的重要性。假如你做了一个简单的网页，其中包含了1个HTML、2个Script脚本、3个CSS、还有10张图片，这个网页成功展示在用户屏幕前，需要完成16次与服务端的交互，由于服务器响应的顺序与发送请求的先后没有直接联系，按照可能出现的响应顺序，一共会有P(16,16) = 20922789888000种可能性。试想一下，如果HTML协议不是设计成无状态的，这16次请求各个有依赖关联，先调用哪一个、先返回哪一个，都会对结果产生影响的话，那协调工作会有多么复杂。

可是，HTTP协议的无状态特性又有悖于我们最常见的网络应用，譬如认证、授权方面，系统总得要获知用户身份才能提供服务，因此，我们也希望HTTP能有一种手段，让服务器至少有办法能够区分出发送请求的用户是谁。为了实现这个目的，[RFC 6265](https://tools.ietf.org/html/rfc6265#section-4.1)规范中定义了HTTP的状态管理机制，在HTTP协议中设计了Set-Cookie指令，该指令的含义是以K/V值对的方式向客户端发送信息，此信息将在此后一定时间内的每次HTTP请求中，以名为Cookie的Header中附带着重新发回服务端，一个典型的Set-Cookie指令如下所示：

```http
Set-Cookie: id=icyfenix; Expires=Wed, 21 Feb 2020 07:28:00 GMT; Secure; HttpOnly
```

从此以后，当客户端对同一个域名（或者Path）的请求中都会带有值对信息“id=icyfenix”，例如以下所示：

```http
GET /index.html HTTP/2.0
Host: icyfenix.cn
Cookie: id=icyfenix; sessionid=38afes7a8
```

根据每次请求传到服务端的Cookie，服务器就能分辨出请求来自于哪一个用户。由于Cookie是放在请求头上的负载（Payload，这个词后面还要频繁用到），不可能存储太大量的数据，放在Cookie中传输也不安全（被窃取，被篡改），所以通常是不会像例子中“id=icyfenix”这样的直接携带数据的。一般来说，Cookie中一般传输的是一个无意义的不重复的字符串，通常以sessionid或者jsessionid为名，服务器拿这个字符串为Key，再在内存中开辟一块空间，以Key/Entity的结构存储每一个在线用户的上下文状态，并辅以一些超时自动清理的管理措施，这种服务端的状态管理机制就是今天大家耳熟能详的Session，Cookie-Session就是在今天广泛应用于大量系统中的、服务端与客户端联动的状态管理机制。

Cookie-Session的方案在本章的主题“安全”上其实多少是占有一定优势的：信息都存储于服务器，不易遭遇传输中被泄漏、篡改的风险，只要通过域保护机制和传输层安全，保证Cookie中的键值不被窃取（如在”漏洞利用“小节中介绍的CSRF、XSS攻击）导致被冒认身份即可。Cookie-Session方案另一大优点是服务端有主动的管理能力，可根据自己的意愿随时修改、清除任意上下文状态，如实现强制某用户下线的功能就很容易。

Session-Cookie在单节点单体服务环境中是非常合适的方案，但当服务能力需要水平扩展，要部署集群时就开始面临一些麻烦了，由于Session建立在服务器的内存中，当服务器水平拓展成多节点时，我们必须在以下三种方案中选择其一：

- 要么就牺牲集群的一致性（Consistency）能力，让均衡器采用亲和式的负载均衡算法（譬如根据用户IP或者sessionid来分配节点），每一个特定用户发出的所有请求都一直被分配到其中某一个节点来提供服务，每个节点都不重复地保存着一部分用户的状态，如果这个节点崩溃了，里面的用户状态便完全丢失。

- 要么就牺牲集群的可用性（Availability）能力，让各个节点之间采用复制式的Session，每一个节点中的Session变动都会发送到组播地址的其他服务器上，这样某个节点崩溃了，不会中断都某个用户的服务，但Session之间组播复制的同步代价高昂，节点越多时越是如此。
- 要么就牺牲集群的分区容错（Partition Tolerance）能力，让普通的服务节点中不再保留状态，将上下文集中放在一个所有服务节点都能访问到的数据节点中进行存储。此时的矛盾是数据节点就成为了单点，一旦数据节点损坏，整个集群都不再能提供服务。（多说一句，现在数据节点常见以Redis来搭建，本身Redis通常也会做集群，但将大集群的CAP问题放到小集群里，并不会让问题的消失，简而言之就是：[禁止套娃](https://www.baidu.com/s?wd=%E7%A6%81%E6%AD%A2%E5%A5%97%E5%A8%83%E6%98%AF%E4%BB%80%E4%B9%88%E6%A2%97)）

以后，我们在微服务架构中还会遇到更多分布式的问题，还会经常受到CAP理论（C、A、P必须牺牲一个）的打击，这是一个很值得深入探讨的技术权衡，但毕竟与本章的“安全”关系不大，这里就不再展开了。现在我只想知道一个问题的答案：前面三种方案都有缺陷，那在分布式应用中，就没有能绕过这些问题的解决方案吗？

我的答案是：有，也没有。如果说要解决分布式环境下的共享数据的CAP矛盾，这是被数学严格证明了不可能的，所以分布式环境中的状态管理一定会受到CAP的限制。但如果是解决分布式下的认证授权问题，那确实还有一些别的法子可想。前面这句话的言外之意是提醒读者，接下来的JWT令牌与Cookie-Session并不是对等的技术方案，它只解决认证授权问题，充其量能携带少量非敏感的信息，只是Cookie-Session在认证授权问题上的替代品，而不会成为Cookie-Session本身的革命者与继承人。

### JWT

前面介绍的Cookie-Session机制在分布式环境下遇到一些问题，在多方系统中，就更不可能谈什么Session层面的数据共享了，而且Cookie也没法跨域。看来，服务器多了，确实不好解决，那就换个思路吧，客户端是唯一的，把数据存储在客户端，每次随着请求发回服务器——JWT就是这种思路的典型代表。

JSON Web Token（JWT），定义于[RFC 7519](https://tools.ietf.org/html/rfc7519)的令牌格式，是目前广泛使用的一种令牌，尤其是与OAuth2配合应用于分布式的、涉及多方的应用系统之中。介绍JWT的具体构成之前，我们先来看一样它是什么样子的，一个JWT的例子如下图所示：

![](./images/jwt.png)

以上截图来自于网站[https://jwt.io/](https://jwt.io/)，当然，数据是我自己编的。左边的是JWT的本体，它通过名为Authorization的Header发送给服务端，前缀是在[RFC 6750](https://tools.ietf.org/html/rfc6750)中定义的bearer，这点在之前关于“认证”的小节中提到过，一个完整的HTTP请求实例如下所示：

```http
GET /restful/products/1 HTTP/1.1
Host: icyfenix.cn
Connection: keep-alive
Authorization: bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJpY3lmZW5peCIsInNjb3BlIjpbIkFMTCJdLCJleHAiOjE1ODQ5NDg5NDcsImF1dGhvcml0aWVzIjpbIlJPTEVfVVNFUiIsIlJPTEVfQURNSU4iXSwianRpIjoiOWQ3NzU4NmEtM2Y0Zi00Y2JiLTk5MjQtZmUyZjc3ZGZhMzNkIiwiY2xpZW50X2lkIjoiYm9va3N0b3JlX2Zyb250ZW5kIiwidXNlcm5hbWUiOiJpY3lmZW5peCJ9.539WMzbjv63wBtx4ytYYw_Fo1ECG_9vsgAn8bheflL8
```

图中右边的内容是经过Base64URL转码之后的令牌明文，是的，明文，JWT令牌默认是不加密的（你自己要加密也行就，接收时自己解密即可）。从明文中可以看到JWT令牌是以JSON结构（毕竟叫JSON Web Token）存储的，结构上可划分为三个部分，每个部分间用点号“.”分隔开。

第一部是**令牌头**（Header），内容如下所示：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

它描述了令牌的类型（统一为typ:JWT）和令牌签名的算法，示例中HS256为HMAC SHA256算法的缩写，其他各种系统所支持的签名算法可以参考[https://jwt.io/](https://jwt.io/)网站所列。

第二部分是**负载**（Payload），是令牌真正需要向服务端传递的信息，在认证问题中，至少应该包括告诉服务端“我是谁”的信息，在授权问题中，至少应该包括告诉服务端“我属于什么角色/权限，有哪些许可”。负载部分是可以完全自定义的，根据具体要解决的问题不同，设计自己所需要的信息（但不能太多，毕竟受HTTP Header大小的限制）。一个JWT负载的示例如下所示：

```json
{
  "username": "icyfenix",
  "authorities": [
    "ROLE_USER",
    "ROLE_ADMIN"
  ],
  "scope": [
    "ALL"
  ],
  "exp": 1584948947,
  "jti": "9d77586a-3f4f-4cbb-9924-fe2f77dfa33d",
  "client_id": "bookstore_frontend"
}
```

而JWT在规范中推荐（无强制约束）了7个字段，如有需要用到这些内容，建议字段名与官方的保持一致：

- iss（Issuer）：签发人
- exp（Expiration Time）：令牌过期时间
- sub（Subject）：主题
- aud （Audience）：令牌受众
- nbf （Not Before）：令牌生效时间
- iat （Issued At）：令牌签发时间
- jti （JWT ID）：令牌编号

第三部分是**签名**（Signature），签名的意思是，使用特定的签名算法（在对象头中公开），使用特定的密钥（Secret，由服务器进行保密，不能公开）对前面两部分内容进行哈希计算，这里密钥实际上承担了加盐（Salt）的作用。以例子中JWT默认的HMAC SHA256算法为例，将通过以下公式产生签名值：

```java
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload) , secret)
```

签名的意义在于确保负载中的信息是可信的、没有被篡改的，也没有在传输过程中丢失。因为被签名的内容哪怕发生了一个字节的变动，也会导致整个签名发生巨大变化。此外，由于这件事情只能由服务端完成（只有它知道Secret），任何人都无法在篡改后重新计算出合法的签名值，所以服务端才能够完全信任客户端传上来的JWT中的负载信息。

在多方系统的应用中，往往还会用非对称算法来进行签名，这时候除了授权服务端持有的可以用于签名的私钥外，还会对其他服务器公开一个公钥，公钥不能用来签名，但是能验证签名是否由私钥所签发的。这样其他服务器也能判断JWT令牌中的信息的真伪。

JWT令牌是多方系统中一种优秀的凭证载体，它不需要任何一个服务节点保留任何一点状态信息，就能够保障认证服务与用户之间的承诺是双方当时真实意图的体现，是准确、完整、不可篡改、且不可抵赖的。同时，由于JWT本身可以携带少量信息，这十分有利于RESTFul API的设计，能够较容易地做成无状态服务，在做水平扩展时就不需要像前面Cookie-Session方案那样考虑如何部署的问题。现实中也确实有一些项目（譬如Fenix's Bookstore）直接采用JWT来承载上下文来实现完全无状态的服务端，这能获得很大的好处，譬如，在你调试Fenix's Bookstore的程序时，随时都可以停止、重启服务端程序，服务重启后客户端仍然是可以毫无感知地继续操作；而服务端有状态的系统，往往就必须通过再次登录、进行前置业务操作来给服务端重建状态。

目前，在大型系统中完全使用JWT来保存上下文状态仍是难以实现的，不过将最热点的服务接口单独抽离出来，做成无状态的、幂等的服务，是一种很有效的提升系统吞吐能力的架构设计。这部分内容将在微服务架构的部分如何划分微服务的章节中进一步探讨。

JWT并不是没有缺点的完美方案，它存在着以下几个明显的缺点：

- **令牌难以主动失效**：JWT令牌一旦签发，理论上就和认证服务器再没有什么瓜葛了，在到期之前就会始终有效，除非服务器部署额外的逻辑，这对某些管理功能的实现是很不利的。譬如，有一种颇为常见的需求是：要求一个用户只能在一台设备上登录，在B设备登陆后，之前已经登录过的A设备就应该自动退出。如果采用JWT，就必须设计一个“黑名单”机制，用来把要主动失效的令牌集中存储起来，而无论这个黑名单是实现在Session、Redis或者数据库中，都会让服务退化回有状态，失去了JWT本身的价值。
- **只能携带相当有限的数据**：HTTP协议并没有强制约束Header的最大长度，但是，各种服务器（甚至是浏览器）都会有约束，譬如Tomcat就要求Header最大不超过8KB，而在Nginx中则默认为4KB，因此在令牌中存储过多的数据不仅浪费带宽，还有出错风险。
- **令牌在客户端如何存储**：这严格来说并不是JWT的问题而是你的问题。如果授权之后操作完了关掉浏览器这是就了解了，那把令牌放到内存里面，哪都不需要存储即可。但不是谁都能忍受一个网站关闭之后下次就一定强制要重新登陆的（大概也就银行的网站可以），那这样的话，把令牌存放到哪里？Cookie、localStorage都有泄漏的可能，而令牌一旦泄漏，就可以冒充你的身份做任何事情。
- **无状态也不总是好处**：这个其实并不算是什么问题。我给你提个需求：你基于JWT的方案，做一个精确的在线用户统计，难搞哦。

我在写这篇文章的时候，在网上搜索资料，发现小小的JWT居然有不少的争议。任何技术都是一种工具而不是银弹，无论是迷信还是[使劲黑它](https://dzone.com/articles/stop-using-jwts-as-session-tokens)，都并无必要。

## 保密 <Badge text="进行中" type="warning"/>

::: tip 保密（Confidentiality）

系统如何保证敏感数据无法被包括系统管理员在内的内外部人员所窃取、滥用？

:::



## 验证 <Badge text="进行中" type="warning"/>

::: tip 验证（Verification）

系统如何确保提交到每项服务中的数据是合乎规则的，不会对系统稳定性、数据一致性、正确性产生风险？

:::





## 传输安全 <Badge text="进行中" type="warning"/>

::: tip 传输安全（Transport Security）

传输安全（Transport Security）：系统如何保证通过网络传输的信息无法被第三方窃听、篡改和冒充？

:::





## 漏洞利用 <Badge text="进行中" type="warning"/>

::: tip 漏洞利用（Exploit）

漏洞利用（Exploit）：系统如何避免在基础设施和应用程序中出现弱点，被攻击者利用？

:::