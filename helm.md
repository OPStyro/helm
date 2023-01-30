# 一、helm的概念

## 1、什么是helm

helm主要是k8s的包管理器，主要用来管理helm中的各种chart包，可以方便的发现、共享和构建k8s应用

相当于yum工具，可以将一个服务相关的所有资源信息整合到一个Chart中，并且可以使用一套资源发布到，像rpm包管理器，可以很方便的将之前打包好的yaml文件部署到k8s上



## 2、传统服务器部署到k8s集群的流程

拉取代码--->打包编译--->准备一堆相关部署的yaml文件(Deployment、ingress、service等等)--->kubectl apply 部署到k8s集群



## 3、helm组件

1. 1. **Chart：**就是helm的一个整合后的chart包，包含一个应用所有的k8s声明模板，类似于yun的rpm包；

​						helm将打包的应用程序部署到k8s，并将他们构建成Chart，这些Chart将所有预配置的应用程序资源以及所有的版本都包含在一个易于管理的包中

​						helm把k8s资源（如deployment，service，ingress等）打包到一个chart中,chart被保chart仓库，通过chart仓库可用来存储和分享chart

1. 1. **Helm客户端：**负责和k8s apiserver通讯
   2. **Repository：**用于发布和存储chart包的仓库，类似yum仓库或docker仓库
   3. **Release：**用chart包部署的一个实例，通过chart在k8s中部署的应用都会产生一个唯一的Release，同一chart部署多次就会产生多个Release

这些yaml部署完成后，他也会记录部署时候的一个版本，维护了一个release版本状态，通过Release这个实例，他会具体的帮我们创建Pod，deployment资源等



## 4、helm目录结构

```bash
# helm create mychart		#创建一个chart，指定chart名：mychart
# tree mychart/
mychart/						#chart包名
|——charts						#存放chart的目录，目录里存放这个chart依赖的所有子chart
|——Chart.yaml					#保存chart的基本信息，包括名字、描述信息及版本等，这个变量文件都可以被templates目录下文件所引用
|——templates					#模板文件目录，目录里面存放所有yaml模板文件，包含了所有部署应用的yaml的文件
|	|——deployment.yaml			#创建deployment对象的模板文件
|	|——helpers.tpl				#放置模板助手的文件，可以在整个chart中重复使用，是放一些templates目录下这些yaml都有可能会用的一些模板
|	|——hpa.yaml
|	|——ingress.yaml
|	|——NOTES.txt				#存放提示信息的文件，介绍chart帮助信息，helm install部署后展示给用户，如何使用chart等，是部署chart后给用户的提示信									息
|	|——serviceaccount.yaml
|	|——service.yaml
│   └── tests					#用于测试的文件，测试完部署chart后，如web，做一个链接，看看是否部署正常
│       └── test-connection.yaml
└── values.yaml					#用于渲染模板的文件（变量文件，自定义变量的值）定义templates目录下的yaml文件可能引用到的变量，values.yaml用于存储templates目录模板文件中用到变量的值，这些变量定义都是为了让templates目录下yaml引用
------------------------------------------------------------------------------------------------------------------------
templates/  #目录用于放置模板文件。当 Tiller 评估 chart 时，它将 templates/ 通过模板渲染引擎发送目录中的所有文件。然后，Tiller 收集这些模板的结果并将				它们发送给 Kubernetes
values.yaml #文件对模板也很重要。该文件包含 chart 默认值。这些值可能在用户在 helm install 或 helm upgrade 期间被覆盖。
Chart.yaml  #文件包含 chart 的说明。可以从模板中查看访问它。该 charts/ 目录可能包含其他 chart（我们称之为子 chart）。在本指南的后面，我们将看到它们在				模板渲染方面如何起作用。
```

## 5、安装helm

GitHub上自己找

## 6、helm的工作原理

Helm管理名为chart的Kubernetes包的工具。Helm可以做以下的事情：

- 从头开始创建新的chart
- 将chart打包成归档(tgz)文件
- 与存储chart的仓库进行交互
- 在现有的Kubernetes集群中安装和卸载chart
- 管理与Helm一起安装的chart的发布周期

**对于Helm，有三个重要的概念：**

1. *chart* 创建Kubernetes应用程序所必需的一组信息。
2. *config* 包含了可以合并到打包的chart中的配置信息，用于创建一个可发布的对象。
3. *release* 是一个与特定配置相结合的chart的运行实例。

### 6.1 组件

Helm是一个可执行文件，执行时分成两个不同的部分：

**Helm客户端** 是终端用户的命令行客户端。负责以下内容：

- 本地chart开发
- 管理仓库
- 管理发布
- 与Helm库建立接口
  - 发送安装的chart
  - 发送升级或卸载现有发布的请求

**Helm库** 提供执行所有Helm操作的逻辑。与Kubernetes API服务交互并提供以下功能：

- 结合chart和配置来构建版本
- 将chart安装到Kubernetes中，并提供后续发布对象
- 与Kubernetes交互升级和卸载chart

独立的Helm库封装了Helm逻辑以便不同的客户端可以使用它。

### 6.2 与k8s通讯

Helm客户端和库是使用Go编程语言编写的

这个库使用Kubernetes客户端库与Kubernetes通信。现在，这个库使用REST+JSON。它将信息存储在Kubernetes的密钥中。 不需要自己的数据库。

如果可能，配置文件是用YAML编写的。

**工作原理架构图：https://www.processon.com/diagraming/6365027ae401fd612f4a6e9b**









# 二、内置对象

## 2.1、内置对象的使用

建议内置函数看官网，比较详细：[Helm | 模板函数列表](https://helm.sh/zh/docs/chart_template_guide/function_list/#date-functions)

对象从模板引擎传递到模板中。你的代码可以传递对象（我们将在说明 `with` 和 `range` 语句时看到示例）。甚至有几种方法在模板中创建新对象，就像我们稍后会看的 `tuple` 函数一样。

对象可以很简单，只有一个值。或者他们可以包含其他对象或函数。例如，`Release` 对象包含多个对象（如 `Release.Name`）并且 `Files` 对象具有一些函数。

- `Release`：这个对象描述了 release 本身。它里面有几个对象：
- `Release.Name`：release 名称
- `Release.Time`：release 的时间
- `Release.Namespace`：release 的 namespace（如果清单未覆盖）
- `Release.Service`：release 服务的名称（始终是 `Tiller`）。
- `Release.Revision`：此 release 的修订版本号。它从 1 开始，每 `helm upgrade` 一次增加一个。
- `Release.IsUpgrade`：如果当前操作是升级或回滚，则将其设置为 `true`。
- `Release.IsInstall`：如果当前操作是安装，则设置为 `true`。
- `Values`：从 `values.yaml` 文件和用户提供的文件传入模板的值。默认情况下，Values 是空的。
- `Chart`：`Chart.yaml` 文件的内容。任何数据 Chart.yaml 将在这里访问。例如 {{.Chart.Name}}-{{.Chart.Version}} 将打印出来 mychart-0.1.0。chart 指南中 [Charts Guide](https://github.com/kubernetes/helm/blob/master/docs/charts.md#the-chartyaml-file) 列出了可用字段
- `Files`：这提供对 chart 中所有非特殊文件的访问。虽然无法使用它来访问模板，但可以使用它来访问 chart 中的其他文件。请参阅 "访问文件" 部分。
- `Files.Get` 是一个按名称获取文件的函数（`.Files.Get config.ini`）
- `Files.GetBytes` 是将文件内容作为字节数组而不是字符串获取的函数。这对于像图片这样的东西很有用。
- `Capabilities`：这提供了关于 Kubernetes 集群支持的功能的信息。
- `Capabilities.APIVersions` 是一组版本信息。
- `Capabilities.APIVersions.Has $version` 指示是否在群集上启用版本（`batch/v1`）。
- `Capabilities.KubeVersion` 提供了查找 Kubernetes 版本的方法。它具有以下值：Major，Minor，GitVersion，GitCommit，GitTreeState，BuildDate，GoVersion，Compiler，和 Platform。
- `Capabilities.TillerVersion` 提供了查找 Tiller 版本的方法。它具有以下值：SemVer，GitCommit，和 GitTreeState。
- `Template`：包含有关正在执行的当前模板的信息
- `Name`：到当前模板的 namespace 文件路径（例如 `mychart/templates/mytemplate.yaml`）
- `BasePath`：当前 chart 模板目录的 namespace 路径（例如 mychart/templates）。

这些值可用于任何顶级模板。我们稍后会看到，这并不意味着它们将在任何地方都要有

内置值始终以大写字母开头。这符合Go的命名约定。当你创建自己的名字时，你可以自由地使用适合你的团队的惯例。一些团队，如Kubernetes chart团队，选择仅使用首字母小写字母来区分本地名称与内置名称。在本指南中，我们遵循该约定。

 {{.Release.Name}}:  前面的  **.**  表示从namespace开始查找



## 2.2、使用的方法：

### 2.2.3、在values.yaml文件中使用

在values.yaml文件中设置一个参数

```yaml
favoriteDrink: coffee
```

在模板中去使用这个参数，包括使用内置对象

​	将 `name:` 硬编码到资源中通常被认为是不好的做法。名称对于 release 应该是唯一的。因此，我们可能希望通过插入 release 名称来生成 name 字段，例如：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
data:
  myvalue: "Hello World"
  drink: {{.Values.favoriteDrink}}
```

### 2.2.4、用命令行来实现赋值

由于 `favoriteDrink` 在默认 `values.yaml` 文件中设置为 `coffee`，这就是模板中显示的值。我们可以轻松地在我们的 helm install 命令中通过加一个 `--set` 添标志来覆盖：

```yaml
helm install --dry-run --debug --set favoriteDrink=slurm ./mychart
SERVER: "localhost:44134"
CHART PATH: /Users/mattbutcher/Code/Go/src/k8s.io/helm/_scratch/mychart
NAME:   solid-vulture
TARGET NAMESPACE:   default
CHART:  mychart 0.1.0
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: solid-vulture-configmap
data:
  myvalue: "Hello World"
  drink: slurm
```

由于 `--set` 比默认 `values.yaml` 文件具有**更高的优先级**，我们的模板生成 `drink: slurm`。

### 2.2.4、多值的使用和赋值

values 文件也可以包含更多结构化内容。例如，我们在 values.yaml 文件中可以创建 `favorite` 部分，然后在其中添加几个键：

```yaml
favorite:
  drink: coffee
  food: pizza
```

现在我们稍微修改模板：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
data:
  myvalue: "Hello World"
  drink: {{.Values.favorite.drink}}			#注意这里
  food: {{.Values.favorite.food}}
```

虽然以这种方式构建数据是可以的，但建议保持 value 树浅一些，平一些。当我们看看为子 chart 分配值时，我们将看到如何使用树结构来命名值。

## 2.3 删除默认key

如果您需要从默认值中删除一个键，可以覆盖该键的值为 null，在这种情况下，Helm 将从覆盖值合并中删除该键。

```bash
比如：helm install ...  --set livenessProbe.httpGet=null
```



例如，stable 版本的 Drupal chart 允许配置 liveness 探测器，如果你配置自定义的 image。以下是默认值：

```yaml
livenessProbe:
  httpGet:
    path: /user/login
    port: http
  initialDelaySeconds: 120
```

如果尝试覆盖 liveness Probe 处理程序 `exec` 而不是 `httpGet`，使用 `--set`

`livenessProbe.exec.command=[cat,docroot/CHANGELOG.txt]`，Helm 会将默认和重写的键合并在一起，**从而产生以下 YAML**：

```yaml
livenessProbe:
  httpGet:
    path: /user/login
    port: http
  exec:
    command:
    - cat
    - docroot/CHANGELOG.txt
  initialDelaySeconds: 120
```

但是，Kubernetes 会报错，因为无法声明多个 liveness Probe 处理程序。为了克服这个问题，你可以指示 Helm 过将 livenessProbe.httpGet 通设置为空来删除它：

```bash
helm install stable/drupal --set image=my-registry/drupal:0.1.0 --set livenessProbe.exec.command=[cat,docroot/CHANGELOG.txt] --set livenessProbe.httpGet=null
```



# 三、模板函数和管道

## 3.1 模板函数

作用：转换模板的数据

当从. Values 对象注入字符串到模板中时，我们引用这些字符串。我们可以通过调用 **quote 模板**指令中的函数来实现：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
data:
  myvalue: "Hello World"
  drink: {{quote .Values.favorite.drink}}
  food: {{quote .Values.favorite.food}}
```

模板函数遵循语法 `functionName arg1 arg2...`。在上面的代码片段中，`quote .Values.favorite.drink` 调用 **quote** 函数并将一个参数传递给它。

Helm 拥有超过 60 种可用函数。其中一些是由 Go 模板语言 [Go template language](https://godoc.org/text/template) 本身定义的。其他大多数都是 Sprig 模板库 [Sprig template library](https://godoc.org/github.com/Masterminds/sprig) 的一部分。在我们讲解例子进行的过程中，我们会看到很多。

## 3.2 管道

表示方法以及使用，反转顺序是模板中的常见做法。你会看到.`val | quote` 比 `quote .val` 更常见。练习也是。

例如：将上面的的变量使用改为用管道的方法来使用

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
data:
  myvalue: "Hello World"
  drink: {{.Values.favorite.drink | quote}}
  food: {{.Values.favorite.food | quote}}
```

> 下面的upper是一个内置函数，意思是变成大写字母，可以是helm官网看看函数的意思

在这个例子中，没有调用 `quote ARGUMENT`，我们调换了顺序。我们使用管道（|）将 “参数” 发送给函数：`.Values.favorite.drink | quote`。使用管道，我们可以将几个功能链接在一起：

```yacas
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
data:
  myvalue: "Hello World"
  drink: {{.Values.favorite.drink | quote}}
  food: {{.Values.favorite.food | upper | quote}}
```

该模板产生的结果如下：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: trendsetting-p-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"				#请注意，我们的原来 pizza 现在已经转换为 "PIZZA"。
```

------

当有像这样管道参数时，第一个评估（`.Values.favorite.drink`）的结果将作为函数的最后一个参数发送。我们可以修改上面的饮料示例来说明一个带有两个参数的函数 `repeat COUNT STRING`：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
data:
  myvalue: "Hello World"
  drink: {{.Values.favorite.drink | repeat 5 | quote}}
  food: {{.Values.favorite.food | upper | quote}}
```

该 repeat 函数将回送给定的字符串和给定的次数，所以我们将得到这个输出：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: melting-porcup-configmap
data:
  myvalue: "Hello World"
  drink: "coffeecoffeecoffeecoffeecoffee"
  food: "PIZZA"
```

## 3.3 使用 default 函数

> 在实际的 chart 中，所有静态默认值应该存在于 values.yaml 中，不应该使用该 default 命令重复（否则它们将是重复多余的）。但是，default 命令对于计算的值是合适的，因为计算值不能在 values.yaml 中声明,下面来看看如何使用default的

value.yaml文件的定义像这样：

```yaml
favorite:
  drink: coffee
  food: pizza
```

经常使用的一个函数是 `default`：`default DEFAULT_VALUE GIVEN_VALUE`。该功能允许在模板内部指定默认值，以防该值被省略。让我们用它来修改上面的饮料示例：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
data:
  myvalue: "Hello World"
  drink: {{.Values.favorite.drink | default "tea" | quote}}
  food: {{.Values.favorite.food | upper | quote}}
```

那么模板产生的结果如下：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: virtuous-mink-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
```

------

如果把value.yaml的文件drink变量删除，例如：

```yaml
favorite:
  #drink: coffee
  food: pizza
```

模板还是和上面的模板一样，现在重新运行 `helm install --dry-run --debug ./mychart` 会产生这个 YAML：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fair-worm-configmap
data:
  myvalue: "Hello World"
  drink: "tea"
  food: "PIZZA"
```

------

在实际的 chart 中，所有静态默认值应该存在于 values.yaml 中，不应该使用该 default 命令重复（否则它们将是重复多余的）。但是，default 命令对于计算的值是合适的，因为计算值不能在 values.yaml 中声明。例如：

```yaml
drink: {{.Values.favorite.drink | default (printf "%s-tea" (include "fullname" .)) }}
```

在一些地方，一个 `if` 条件可能比这 `default` 更适合。我们将在下一节中看到这些。

模板函数和管道是转换信息并将其插入到 YAML 中的强大方法。但有时候需要添加一些比插入字符串更复杂一些的模板逻辑。在下一节中，我们将看看模板语言提供的控制结构。

## 3.4 运算符函数

对于模板，运算符（eq，ne，lt，gt，and，or 等等）都是已实现的功能。在管道中，运算符可以用圆括号（`(` 和 `)`）分组。

将运算符放到声明的前面，后面跟着它的参数，就像使用函数一样。要多个运算符一起使用，将每个函数通过圆括号分隔

```yaml
{{/* include the body of this if statement when the variable .Values.fooString xists and is set to "foo" */}}
{{if and .Values.fooString (eq .Values.fooString "foo") }}
    {{...}}
{{end}}


{{/* do not include the body of this if statement because unset variables evaluate o false and .Values.setVariable was negated with the not function. */}}
{{if or .Values.anUnsetVariable (not .Values.aSetVariable) }}
   {{...}}
{{end}}
```

# 四、流控制

控制结构（模板说法中称为 “动作”）为模板作者提供了控制模板生成流程的能力。Helm 的模板语言提供了以下控制结构：这节主要描述前三个：

- `if/else` 用于创建条件块
- `with` 指定范围
- `range`，它提供了一个 “for each” 风格的循环

除此之外，它还提供了一些声明和使用命名模板段的操作：

- `define` 在模板中声明一个新的命名模板
- `template` 导入一个命名模板
- `block` 声明了一种特殊的可填写模板区域

## 4.1 if/else

语法：

```yaml
**{{- if PIPELINE }}
  # Do something*
**{{- else if OTHER PIPELINE }}
  # Do something else*
**{{- else }}
  # Default case*
**{{- end }}
```

> 注意在YAML中有一个空行，为什么？当模板引擎运行时，它 *移除了* `{{` 和 `}}` 里面的内容，但是留下的空白完全保持原样。
>
> YAML认为空白是有意义的，因此管理空白变得很重要。幸运的是，Helm模板有些工具可以处理此类问题。
>
> 首先，模板声明的大括号语法可以通过特殊的字符修改，并通知模板引擎取消空白。`{{- `(包括添加的横杠和空格)表示向左删除空白， 而` -}}`表示右边的空格应该被去掉。 *一定注意空格就是换行*
>
> 要确保`-`和其他命令之间有一个空格。 `{{- 3 }}` 表示“删除左边空格并打印3”，而`{{-3 }}`表示“打印-3”。

**具体的语句结构看控制空格**

------

注意我们讨论的是 *管道* 而不是值。这样做的原因是要清楚地说明控制结构可以执行整个管道，而不仅仅是计算一个值。

如果是以下值时，管道会被设置为 *false*：

- 布尔false
- 数字0
- 空字符串
- `nil` (空或null)
- 空集合(`map`, `slice`, `tuple`, `dict`, `array`)

在所有其他条件下，条件都为true。

------

value.yaml文件还是这个：

```yaml
favorite:
  #drink: coffee
  food: pizza
```

为 ConfigMap 模板添加一个简单的条件。如果饮料被设置为咖啡，我们将添加另一个设置：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
data:
  myvalue: "Hello World"
  drink: {{.Values.favorite.drink | default "tea" | quote}}
  food: {{.Values.favorite.food | upper | quote}}
  {{if and .Values.favorite.drink (eq .Values.favorite.drink "coffee") }}mug: true{{ end }}
```

> 注意 `.Values.favorite.drink` 必须已定义，否则在将它与 “coffee” 进行比较时会抛出错误。由于我们在上一个例子中注释掉了 `drink：coffee`，因此输出不应该包含 `mug：true` 标志。但是如果我们将该行添加回 `values.yaml` 文件中，输出应该如下所示:

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: eyewitness-elk-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  mug: true
```



## 4.2 控制空格

为了在模板中使用条件控制语句更加的美观易读，空格的管理更加有效，例子如下：

### 4.2.1  第一个表示方法 [-]

使用这个语法，我们就可修改我们的模板，去掉新加的空白行：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
  {{- if eq .Values.favorite.drink "coffee" }}			#注意这一行
  mug: "true"
  {{- end }}
```

### 4.2.2  第二个表示方法 [*]

调整上述内容，用一个`*`来代替每个遵循此规则被删除的空白， 在行尾的`*`表示删除新行的字符：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}*			#注意这里
**{{- if eq .Values.favorite.drink "coffee" }}
  mug: "true"*
**{{- end }}
```

### 4.2.3  注意最后面的 【-】

要注意这个删除字符的更改，很容易意外地出现情况：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
  {{- if eq .Values.favorite.drink "coffee" -}}			#注意这一行
  mug: "true"
  {{- end -}}
```

这样产生的结果：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: eyewitness-elk-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"mug: true			#注意这里
```

最终，有时这更容易告诉模板系统如何缩进，而不是试图控制模板指令间的间距。因此，您有时会发现使用`indent`方法(`{{ indent 2 "mug:true" }}`)会很有用。

## 4.3 修改使用`with`的范围

下一个控制结构是`with`操作。这个用来控制变量范围。回想一下，`.`是对 *当前作用域* 的引用。因此 `.Values`就是告诉模板在当前作用域查找`Values`对象。

with的语法与 if 语句相似：

```bash
{{ with PIPLINE}}
	# restricted scope
{{ end }}
```

------

### 4.3.1 wiht 作用域的使用

如何使用with

作用域可以被改变。`with`允许你为特定对象设定当前作用域(`.`)。比如，我们已经在使用`.Values.favorite`。 修改配置映射中的`.`的作用域指向`.Values.favorite`：

例如：由 if 改为 with

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
```

**注意：`with`后面的块只有在 `PIPELINE` 的值不为空时才会执行。**

**错误示例：**

注意现在我们可以引用.drink和.food了，而不必限定他们。因为with语句设置了.指向.Values.favorite。 .被重置为{{ end }}之后的上一个作用域。

```http
# 但是这里有个注意事项，在限定的作用域内，无法使用.访问父作用域的对象。错误示例如下：
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ .Release.Name }}				# 注意这里：在限定的作用域内，无法使用.访问父作用域的对象,要使用的话可以加上 $ :release: {{ $.Release.Name }}
  {{- end }}
  
这样会报错因为Release.Name不在.限定的作用域内。但是如果对调最后两行就是正常的， 因为在{{ end }}之后作用域被重置了
```

------

**以下是正确的实例：**

​		1、把 with 限定的父域 end 重置之后才可以被访问其他父域

```yaml
{{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
  release: {{ .Release.Name }}
```

​		2、可以使用`$`从父作用域中访问`Release.Name`对象。当模板开始执行后`$`会被映射到根作用域，且执行过程中不会更改。 下面这种方式也可以正常工作：

```yaml
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ $.Release.Name }}
  {{- end }}
```

在介绍了`range`之后，我们会看看模板变量，提供了上述作用域问题的另一种解决方案。

## 4.4 rang操作循环

很多编程语言支持使用`for`循环，`foreach`循环，或者类似的方法机制。 在Helm的模板语言中，在一个集合中迭代的方式是使用`range`操作符。

**实例：**rang 的使用

方法1：

先编写一个 value.yaml 文件

```yaml
favorite:
  drink: coffee
  food: pizza
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions
```

在编写一个模板 ，`pizzaToppings`列表（模板中称为切片）。修改模板把这个列表打印到配置映射中：

titile 是内置变量，首字母转换成大写，

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
  toppings: |-
    {{- range .Values.pizzaToppings }}
    - {{ . | title | quote }}
    {{- end }} 
```

方法2：

我可以使用`$`从父作用域访问`Values.pizzaToppings`列表。当模板开始执行后`$`会被映射到根作用域， 且执行过程中不会更改。下面这种方式也可以正常工作：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  toppings: |-
    {{- range $.Values.pizzaToppings }}
    - {{ . | title | quote }}
    {{- end }}    
  {{- end }}
```

仔细看看`toppings:`列表。`range`方法“涵盖”（迭代）`pizzaToppings`列表。但现在发生了有意思的事情。 就像`with`设置了`.`的作用域，`range`操作符也做了同样的事。每一次循环，`.`都会设置为当前的披萨配料。 也就是说，第一次`.`设置成了`mushrooms`，第二次迭代设置成了`cheese`，等等。

可以直接发送`.`的值给管道，因此当我们执行`{{ . | title | quote }}`时，它会发送`.`到`title`然后发送到`quote`。 如果执行这个模板，输出是这样的：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: edgy-dragonfly-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  toppings: |-
    - "Mushrooms"
    - "Cheese"
    - "Peppers"
    - "Onions"   
```

> 正如例子中所示，`|-`标识在YAML中是指多行字符串。这在清单列表中嵌入大块数据是很有用的技术。

方法3：

现在，我们已经处理了一些棘手的事情。`toppings: |-`行是声明的多行字符串。所以这个配料列表实际上不是YAML列表， 是个大字符串。为什么要这样做？因为在配置映射`data`中的数据是由键值对组成，key和value都是简单的字符串。 要理解这个示例，请查看 [Kubernetes ConfigMap 文档](https://kubernetes.io/docs/user-guide/configmap/)。 但对于我们来说，这个细节并不重要。

有时能在模板中快速创建列表然后迭代很有用，Helm模板的`tuple`可以很容易实现该功能。在计算机科学中， 元组表示一个有固定大小的类似列表的集合，但可以是任意数据类型。这大致表达了`tuple`的用法。

```yaml
sizes: |-
    {{- range tuple "small" "medium" "large" }}
    - {{ . }}
    {{- end }}    
```

上述模板会生成以下内容：

```yaml
 sizes: |-
    - small
    - medium
    - large    
```

除了列表和元组，`range`可被用于迭代有键值对的集合（像`map`或`dict`）。我们会在下一部分介绍模板变量是看到它是如何应用的。

# 五、变量

函数、管道符、对象和控制结构都可以控制，我们转向很多编程语言中更基本的思想之一：变量。 在模板中，很少被使用。但是我们可以使用变量简化代码，并更好地使用`with`和`range`。

在之前的例子中，我们看到下面的代码会失败：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ .Release.Name }}
  {{- end }}
```

`Release.Name` 不在`with`块的限制范围内。解决作用域问题的一种方法是将对象分配给可以不考虑当前作用域而访问的变量。

------

Helm模板中，变量是对另一个对象的命名引用。遵循`$name`变量的格式且指定了一个特殊的赋值运算符：`:=`。 我们可以使用针对`Release.Name`的变量重写上述内容。

## 5.1 with 中的跨域访问

with 限定了指定域，如何在 with 里面实现跨域访问

**用变量：**在 with 执行之前用变量存下需要访问的跨域，例如：

**value.yaml 文件**

```yaml
favorite:
  drink: coffee
  food: pizza
```



Helm模板中，变量是对另一个对象的命名引用。遵循`$name`变量的格式且指定了一个特殊的赋值运算符：`:=`。 我们可以使用针对`Release.Name`的变量重写上述内容。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- $relname := .Release.Name -}}
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ $relname }}
  {{- end }}
```

运行之后会生成以下内容：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: viable-badger-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  release: viable-badger
```

## 5.2 变量在range中的使用

### 5.2.1 捕获索引和值

**value.yaml文件**

```yaml
favorite:
  drink: coffee
  food: pizza
```

**模板文件**

变量在`range`循环中特别有用。可以用于类似列表的对象，以捕获索引和值：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-configmap
data:
  myvalue: "Hello World"
  toppings: |-
    {{- range $index, $topping := .Values.pizzaToppings }}
      {{ $index }}: {{ $topping }}
    {{- end }}   
```

注意先是`range`，然后是变量，然后是赋值运算符，然后是列表。会将整型索引（从0开始）赋值给`$index`并将值赋值给`$topping`。 执行会生成：

```yaml
 toppings: |-
      0: mushrooms
      1: cheese
      2: peppers
      3: onions      
```



### 5.2.2 获取key和value

对于数据结构有key和value，可以使用`range`获取key和value。比如，可以通过`.Values.favorite`进行循环：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
```

第一次迭代，`$key`会是`drink`且`$val`会是`coffee`，第二次迭代`$key`会是`food`且`$val`会是`pizza`。 运行之后会生成：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: eager-rabbit-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
```

## 5.3 $ 全局变量

变量一般不是"全局的"。作用域是其声明所在的块。上面我们在模板的顶层赋值了`$relname`。变量的作用域会是整个模板。 但在最后一个例子中`$key`和`$val`作用域会在`{{ range... }}{{ end }}`块内。

但有个变量一直是全局的 - `$` - 这个变量一直是指向根的上下文。当在一个范围内循环时会很有用，同时你要知道chart的版本名称。

例如：

```yaml
{{- range .Values.tlsSecrets }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ .name }}
  labels:
    # Many helm templates would use `.` below, but that will not work,
    # however `$` will work here
    app.kubernetes.io/name: {{ template "fullname" $ }}
    # I cannot reference .Chart.Name, but I can do $.Chart.Name
    helm.sh/chart: "{{ $.Chart.Name }}-{{ $.Chart.Version }}"
    app.kubernetes.io/instance: "{{ $.Release.Name }}"
    # Value from appVersion in Chart.yaml
    app.kubernetes.io/version: "{{ $.Chart.AppVersion }}"
    app.kubernetes.io/managed-by: "{{ $.Release.Service }}"
type: kubernetes.io/tls
data:
  tls.crt: {{ .certificate }}
  tls.key: {{ .key }}
---
{{- end }}
```

到目前为止，我们只看到在一个文件声明的一个模板。但是Helm模板语言一个很强大的特性是能够声明多个模板并将它们一起使用。 我们将在下一节讨论这个问题。

## 5.4 跨域访问总结

**1、if / else 控制语句实现跨域**，不会限定域名内

2、**with** 中限定域，实现跨域访问

方法1：用变量

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- $relname := .Release.Name -}}
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ $relname }}
  {{- end }}
```

方法2：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ $.Release.Name }}
  {{- end }}
```



# 六、命名模板

此时需要越过模板，开始创建其他内容了。该部分我们会看到如何在一个文件中定义 *命名模板*，并在其他地方使用。*命名模板* (有时称作一个 *部分* 或一个 *子模板*)仅仅是在文件内部定义的模板，并使用了一个名字。有两种创建方式和几种不同的使用方法

在 [流控制](https://helm.sh/zh/docs/chart_template_guide/control_structures)部分， 我们介绍了三种声明和管理模板的方法：`define`，`template`，和`block`。在这部分，我们将使用这三种操作并介绍一种特殊用途的 `include`方法，类似于`template`操作。

命名模板时要记住一个重要细节：**模板名称是全局的**。如果您想声明两个相同名称的模板，哪个最后加载就使用哪个。 因为在子chart中的模板和顶层模板一起编译，命名时要注意 *chart特定名称*。

一个常见的命名惯例是用chart名称作为模板前缀：`{{ define "mychart.labels" }}`。使用特定chart名称作为前缀可以避免可能因为 两个不同chart使用了相同名称的模板而引起的冲突。

## 6.1 局部的和`_`文件

目前为止，我们已经使用了单个文件，且单个文件中包含了单个模板。但Helm的模板语言允许你创建命名的嵌入式模板， 这样就可以在其他位置按名称访问。

在编写模板细节之前，文件的命名惯例需要注意：

- `templates/`中的大多数文件被视为包含Kubernetes清单
- `NOTES.txt`是个例外
- 命名以下划线(`_`)开始的文件则假定 *没有* 包含清单内容。这些文件不会渲染为Kubernetes对象定义，但在其他chart模板中都可用。

这些文件用来存储局部和辅助对象，实际上当我们第一次创建`mychart`时，会看到一个名为`_helpers.tpl`的文件，这个文件是模板局部的默认位置。

## 6.2 用`define`和`template`声明和使用模板

`define`操作允许我们在模板文件中创建一个命名模板，语法如下：

```yaml
{{- define "MY.NAME" }}
  # body of template here
{{- end }}
```

例如：定义一个模板封装Kubernetes的标签：

```yaml
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}			# now 内置函数，当前时间；htmlDate 格式化插入到HTML日期选择器输入字段的日期。
{{- end }}
```

将模板嵌入到了已有的配置映射中，然后使用`template`包含进来：

```yaml
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
{{- end }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
```

当模板引擎读取该文件时，它会存储`mychart.labels`的引用直到`template "mychart.labels"`被调用。 然后会按行渲染模板，因此结果类似这样：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: running-panda-configmap
  labels:
    generator: helm
    date: 2016-11-02
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
```

注意：`define`不会有输出，除非像本示例一样用模板调用它。

按照惯例`define`方法会有个简单的文档块(`{{/* ... */}}`)来描述要做的事。

尽管这个定义是在`_helpers.tpl`中，但它仍能访问`configmap.yaml`：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
```

如上所述，**模板名称是全局的**。因此，如果两个模板使用相同名字声明，会使用最后出现的那个。由于子chart中的模板和顶层模板一起编译， 最好用 *chart特定名称* 命名你的模板。常用的命名规则是用chart的名字作为模板的前缀： `{{ define "mychart.labels" }}`。

## 6.3 设置模板的范围

在上面定义的模板中，我们没有使用任何对象，仅仅使用了方法。修改定义好的模板让其包含chart名称和版本号：

```yaml
{{/* Generate basic labels */}}
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
    chart: {{ .Chart.Name }}
    version: {{ .Chart.Version }}
{{- end }}
```

> 如果渲染这个，会得到以下错误：

```http
$ helm install --dry-run moldy-jaguar ./mychart
Error: unable to build kubernetes objects from release manifest: error validating "": error validating data: [unknown object type "nil" in ConfigMap.metadata.labels.chart, unknown object type "nil" in ConfigMap.metadata.labels.version]
```



要查看渲染了什么，可以用`--disable-openapi-validation`参数重新执行： `helm install --dry-run --disable-openapi-validation moldy-jaguar ./mychart`。 结果并不是我们想要的：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: moldy-jaguar-configmap
  labels:
    generator: helm
    date: 2021-03-06
    chart:
    version:
```

名字和版本号怎么了？没有出现在我们定义的模板中。当一个（使用`define`创建的）命名模板被渲染时，会接收被`template`调用传入的内容。 在我们的示例中，包含模板如下：

```yaml
{{- template "mychart.labels" }}
```

没有内容传入，所以模板中无法用`.`访问任何内容。但这个很容易解决，只需要传递一个范围给模板：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" . }}
```

注意这个在`template`调用末尾传入的`.`，我们可以简单传入`.Values`或`.Values.favorite`或其他需要的范围。但一定要是顶层范围。

现在我们可以用`helm install --dry-run --debug plinking-anaco ./mychart`执行模板，然后得到：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: plinking-anaco-configmap
  labels:
    generator: helm
    date: 2021-03-06
    chart: mychart
    version: 0.1.0
```

现在`{{ .Chart.Name }}`解析为`mychart`，`{{ .Chart.Version }}`解析为`0.1.0`。

## 6.4 include 方法

一个简单的模板如下：

```yaml
{{- define "mychart.app -"}}
app_name: {{ .Chart.name }}
app_version: "{{ .Chart.Version }}"
```

把这个插入到模板的`labels:`部分和`data:`部分：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  labels:
    {{ template "mychart.app" . }}
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
{{ template "mychart.app" . }}
```

如果渲染这个，会得到以下错误：

```http
$ helm install --dry-run measly-whippet ./mychart
Error: unable to build kubernetes objects from release manifest: error validating "": error validating data: [ValidationError(ConfigMap): unknown field "app_name" in io.k8s.api.core.v1.ConfigMap, ValidationError(ConfigMap): unknown field "app_version" in io.k8s.api.core.v1.ConfigMap]
```

要查看渲染了什么，可以用`--disable-openapi-validation`参数重新执行： `helm install --dry-run --disable-openapi-validation measly-whippet ./mychart`。 输入不是我们想要的：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: measly-whippet-configmap
  labels:
    app_name: mychart
app_version: "0.1.0"			# 缩进不对
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
app_name: mychart				# 缩进不对
app_version: "0.1.0"			# 缩进不对，可以使用 indent 2 来进行缩进，实例如下
```

注意两处的`app_version`缩进都不对，为啥？因为被替换的模板中文本是左对齐的。由于`template`是一个行为，不是方法，无法将 `template`调用的输出传给其他方法，数据只是简单地按行插入。

**使用`indent`正确地缩进了`mychart.app`模板：**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  labels:
{{ include "mychart.app" . | indent 4 }}		# 注意这里
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
{{ include "mychart.app" . | indent 2 }}		# 注意这里
```

现在生成的YAML每一部分都可以正确缩进了：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: edgy-mole-configmap
  labels:
    app_name: mychart
    app_version: "0.1.0"
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
  app_name: mychart
  app_version: "0.1.0"
```

> 相较于使用`template`，在helm中使用`include`被认为是更好的方式 只是为了更好地处理YAML文档的输出格式

有时我们需要导入内容，但不是作为模板，也就是按字面意义导入文件内容，可以通过使用`.Files`对象访问文件来实现， 这将在下一部分展开描述。

# 七、在模板内部访问文件

在上一节中，我们研究了几种创建和访问模板的方法。这样可以很容易从一个模板导入到另一个模板中。 但有时想导入的是不是模板的文件并注入其内容，而无需通过模板渲染发送内容。

Helm 提供了通过`.Files`对象访问文件的方法。不过，在我们使用模板示例之前，有些事情需要注意：

- 可以添加额外的文件到chart中。虽然这些文件会被绑定。但是要小心，由于Kubernetes对象的限制，Chart必须小于1M。
- 通常处于安全考虑，一些文件无法通过 **.Files** 对象访问：
  - 无法访问`templates/`中的文件
  - 无法访问使用`.helmignore`排除的文件
  - helm应用 [subchart](https://helm.sh/zh/docs/chart_template_guide/subcharts_and_globals)之外的文件，包括父级中的，不能被访问的

- Chart不能保留UNIX模式信息，因此当文件涉及到`.Files`对象时，文件级权限不会影响文件的可用性。

- [基本示例](https://helm.sh/zh/docs/chart_template_guide/accessing_files/#basic-example)
- [Path辅助对象](https://helm.sh/zh/docs/chart_template_guide/accessing_files/#path-helpers)
- [全局模式](https://helm.sh/zh/docs/chart_template_guide/accessing_files/#glob-patterns)
- [ConfigMap和密钥的实用功能](https://helm.sh/zh/docs/chart_template_guide/accessing_files/#configmap-and-secrets-utility-functions)
- [编码](https://helm.sh/zh/docs/chart_template_guide/accessing_files/#encoding)
- [文件行](https://helm.sh/zh/docs/chart_template_guide/accessing_files/#lines)

## 7.1 Basic example 基本示例：

先不管警告，我们来写一个读取三个文件到配置映射ConfigMap的模板。开始之前，我们会在chart中添加三个文件， 直接放到`mychart/`目录中。

`config1.toml`:

```toml
message = Hello from config 1
```

`config2.toml`:

```toml
message = This is config 2	
```

`config3.toml`:

```toml
message = Goodbye from config 3
```

每个都是简单的TOML文件（类似于windows老式的INI文件）。我们知道这些文件的名称，因此我们使用`range`功能遍历它们并将它们的内容注入到我们的ConfigMap中。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- $files := .Files }}
  {{- range tuple "config1.toml" "config2.toml" "config3.toml" }}		# tuple：存放元组
  {{ . }}: |-					# 打印文件名
        {{ $files.Get . }}		 # 获取改文件名的文件的内容
  {{- end }}
##########################################################################
这个配置映射使用了之前章节讨论过的技术。比如，我们创建了一个$files变量来引用.Files对象。我们也使用了tuple方法创建了一个可遍历的文件列表。 然后我们打印每个文件的名字({{ . }}: |-)，然后通过{{ $files.Get . }}打印文件内容。
```

执行这个模板会生成包含了三个文件所有内容的单个配置映射：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: quieting-giraf-configmap
data:
  config1.toml: |-
        message = Hello from config 1

  config2.toml: |-
        message = This is config 2

  config3.toml: |-
        message = Goodbye from config 3
```

## 72 Path helpers 辅助对象

使用文件时，对文件路径本身执行一些标准操作会很有用。为了实现这些，Helm从Go的 [path](https://golang.org/pkg/path/)包中导入了一些功能。 都使用了与Go包中一样的名称就可以访问。但是第一个字符使用了小写，比如`Base`变成了`base`等等。

**导入的功能包括：**

- Base
- Dir
- Ext
- IsAbs
- Clean

## 7.3 Glob patterns

当你的chart不断变大时【一个目录里面有好几种文件】，你会发现你强烈需要组织你的文件，所以我们提供了一个 `Files.Glob(pattern string)`方法来使用 [全局模式](https://godoc.org/github.com/gobwas/glob)的灵活性读取特定文件。

**`.Glob`返回一个`Files`类型，因此你可以在返回对象上调用任意的`Files`方法。**

比如实例： 目录结构

```
foo/:
  foo.txt foo.yaml

bar/:
  bar.go bar.conf baz.yaml
```

全局模式下您有多种选择：

```yaml
{{ $currentScope := .}}
{{ range $path, $_ :=  .Files.Glob  "**.yaml" }}	# .Files.Glob  "**.yaml"：全局匹配yaml文件的文件名
    {{- with $currentScope}}
        {{ .Files.Get $path }}
    {{- end }}
{{ end }}
```

Or

```yaml
{{ range $path, $_ :=  .Files.Glob  "**.yaml" }}	# .Files.Glob  "**.yaml"：全局匹配yaml文件的文件名
      {{ $.Files.Get $path }}
{{ end }}
```

## 7.4 ConfigMap and Secrets utility functions

（在Helm 2.0.2及后续版本可用）

把文件内容放入配置映射和密钥是很普遍的功能，为了运行时挂载到你的pod上。为了实现它，我们提供了一些基于`Files`类型的实用方法。

为了进一步组织文件，这些方法结合`Glob`方法使用时尤其有用。

上面的文件结构使用 [Glob](https://helm.sh/zh/docs/chart_template_guide/accessing_files/#glob-patterns)时的示例如下：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: conf
data:
{{ (.Files.Glob "foo/*").AsConfig | indent 2 }}	# (.Files.Glob "foo/*").AsConfig：使用YAML格式返回文件体的方法
---
apiVersion: v1
kind: Secret
metadata:
  name: very-secret
type: Opaque
data:
{{ (.Files.Glob "bar/*").AsSecrets | indent 2 }}	#  (.Files.Glob "bar/*").AsSecrets：使用Base 64编码字符串返回文件体的方法
```

## 7.5 Encoding 编码

您可以导入一个文件并使用模板的base-64方式对其进行编码来保证成功传输：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-secret
type: Opaque
data:
  token: |-
        {{ .Files.Get "config1.toml" | b64enc }}
####################################################
Helm有以下编码和解码函数：
	b64enc/b64dec: 编码或解码 Base64
```

上面的内容使用我们之前使用的相同的`config1.toml`文件进行编码：

```yaml
# Source: mychart/templates/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: lucky-turkey-secret
type: Opaque
data:
  token: |-
        bWVzc2FnZSA9IEhlbGxvIGZyb20gY29uZmlnIDEK
```



## 7.6 Lines

有时需要访问模板中的文件的每一行。我们提供了一个方便的`Lines`方法。

你可以使用`range`方法遍历`Lines`：

```yaml
data:
  some-file.txt: {{ range .Files.Lines "foo/bar.txt" }}
    {{ . }}{{ end }}
##########################################################
Files.Lines 逐行读取文件内容的方法。迭代文件中每一行时很有用
```

在`helm install`过程中无法将文件传递到chart外。因此如果你想请求用户提供数据，必须使用`helm install -f`或`helm install --set`加载。

该部分讨论整合了我们对编写Helm模板的工具和技术的深入研究。下个章节我们会看到如何使用特殊文件`templates/NOTES.txt`， 向chart的用户发送安装后的说明。

# 八、创建一个NOTES.txt文件

该部分会介绍为chart用户提供说明的Helm工具。在`helm install` 或 `helm upgrade`命令的最后，Helm会打印出对用户有用的信息。 使用模板可以高度自定义这部分信息。

要在chart添加安装说明，只需创建`templates/NOTES.txt`文件即可。该文件是纯文本，但会像模板一样处理， 所有正常的模板函数和对象都是可用的。

**让我们创建一个简单的`NOTES.txt`文件：**

```
Thank you for installing {{ .Chart.Name }}.

Your release is named {{ .Release.Name }}.

To learn more about the release, try:

  $ helm status {{ .Release.Name }}
  $ helm get all {{ .Release.Name }}
```

现在如果我们执行`helm install rude-cardinal ./mychart` 会在底部看到：

```
RESOURCES:
==> v1/Secret
NAME                   TYPE      DATA      AGE
rude-cardinal-secret   Opaque    1         0s

==> v1/ConfigMap
NAME                      DATA      AGE
rude-cardinal-configmap   3         0s


NOTES:
Thank you for installing mychart.

Your release is named rude-cardinal.

To learn more about the release, try:

  $ helm status rude-cardinal
  $ helm get all rude-cardinal
```

使用`NOTES.txt`这种方式是给用户提供关于如何使用新安装的chart细节信息的好方法。尽管并不是必需的，强烈建议创建一个`NOTES.txt`文件。

# 九、子chart和全局值

到目前为止，我们只使用了一个chart。但chart可以使用依赖，称为 *子chart*，且有自己的值和模板。 该章节我们会创建一个子chart并能看到访问模板中的值的不同方式。

在深入研究代码之前，需要了解一些应用的子chart的重要细节：

1. 子chart被认为是“独立的”，意味着子chart从来不会显示依赖它的父chart。
2. 因此，子chart无法访问父chart的值。
3. 父chart可以覆盖子chart的值。
4. Helm有一个 *全局值* 的概念，所有的chart都可以访问。

> 这些限制不一定都适用于提供标准化辅助功能的 [library charts](https://helm.sh/zh/docs/topics/library_charts)。

浏览本节的示例之后，这些概念会变得更加清晰。

## 9.1 创建chart

为了做这些练习，我们可以从本指南开始时创建的`mychart/`开始，并在其中添加一个新的chart。

```bash
$ cd mychart/charts
$ helm create mysubchart
Creating mysubchart
$ rm -rf mysubchart/templates/*
```

注意，和以前一样，我们删除了所有的基本模板，然后从头开始，在这个指南中，我们聚焦于模板如何工作，而不是管理依赖。 但 [Chart指南](https://helm.sh/zh/docs/topics/charts)提供了更多子chart运行的信息。

## 9.2 在子chart中添加值和模板

下一步，为`mysubchart`创建一个简单的模板和values文件。`mychart/charts/mysubchart`应该已经有一个`values.yaml`。 设置如下：

```yaml
dessert: cake
```

下一步，在`mychart/charts/mysubchart/templates/configmap.yaml`中创建一个新的配置映射模板：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-cfgmap2
data:
  dessert: {{ .Values.dessert }}
```

因为每个子chart都是 *独立的chart*，可以单独测试`mysubchart`：

```yaml
$ helm install --generate-name --dry-run --debug mychart/charts/mysubchart
SERVER: "localhost:44134"
CHART PATH: /Users/mattbutcher/Code/Go/src/helm.sh/helm/_scratch/mychart/charts/mysubchart
NAME:   newbie-elk
TARGET NAMESPACE:   default
CHART:  mysubchart 0.1.0
MANIFEST:
---
# Source: mysubchart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: newbie-elk-cfgmap2
data:
  dessert: cake
```

## 9.3 用父chart的值来覆盖

原始chart，`mychart`现在是`mysubchart`的 *父*。这种关系是基于`mysubchart`在`mychart/charts`中这一事实。

因为`mychart`是父级，可以在`mychart`指定配置并将配置推送到`mysubchart`。比如可以修改`mychart/values.yaml`如下：

```yaml
favorite:
  drink: coffee
  food: pizza
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions

mysubchart:
  dessert: ice cream
```

注意最后两行，在`mysubchart`中的所有指令会被发送到`mysubchart`chart中。因此如果运行`helm install --dry-run --debug mychart`，会看到一项`mysubchart`的配置：

```yaml
# Source: mychart/charts/mysubchart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: unhinged-bee-cfgmap2
data:
  dessert: ice cream
```

现在，子chart的值已经被顶层的值覆盖了。

这里需要注意个重要细节。我们不会改变`mychart/charts/mysubchart/templates/configmap.yaml`模板到 `.Values.mysubchart.dessert`的指向。从模板的角度来看，值依然是在`.Values.dessert`。当模板引擎传递值时，会设置范围。 因此对于`mysubchart`模板，`.Values`中只提供专门用于`mysubchart`的值。

但是有时确实希望某些值对所有模板都可用。这是使用全局chart值完成的。

## 9.4 全局Chart值

全局值是使用完全一样的名字在所有的chart及子chart中都能访问的值。全局变量需要显示声明。不能将现有的非全局值作为全局值使用。

这些值数据类型有个保留部分叫`Values.global`，可以用来设置全局值。在`mychart/values.yaml`文件中设置一个值如下：

```yaml
favorite:
  drink: coffee
  food: pizza
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions

mysubchart:
  dessert: ice cream

global:
  salad: caesar
```

因为全局的工作方式，`mychart/templates/configmap.yaml`和`mysubchart/templates/configmap.yaml` 应该都能以`{{ .Values.global.salad }}`进行访问。

mychart/templates/configmap.yaml:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  salad: {{ .Values.global.salad }}
```

`mysubchart/templates/configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-cfgmap2
data:
  dessert: {{ .Values.dessert }}
  salad: {{ .Values.global.salad }}
```

现在如果预安装，两个输出会看到相同的值：

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: silly-snake-configmap
data:
  salad: caesar

---
# Source: mychart/charts/mysubchart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: silly-snake-cfgmap2
data:
  dessert: ice cream
  salad: caesar
```

## 9.5 与子chart共享模板

父chart和子chart可以共享模板。在任意chart中定义的块在其他chart中也是可用的。

比如，我们可以这样定义一个简单的模板：

```yaml
{{- define "labels" }}from: mychart{{ end }}
```

回想一下模板标签时如何 *全局共享的*。因此，`标签`chart可以包含在任何其他chart中。

当chart开发者在`include` 和 `template` 之间选择时，使用`include`的一个优势是`include`可以动态引用模板：

```yaml
{{ include $mytemplate }}
```

## 9.6 避免使用块

Go 模板语言提供了一个 `block` 关键字允许开发者提供一个稍后会被重写的默认实现。在Helm chart中， 块并不是用于覆盖的最好工具，因为如果提供了同一个块的多个实现，无法预测哪个会被选定。

建议改为使用`include`。

# 十、.helmignore 文件

`.helmignore` 文件用来指定你不想包含在你的helm chart中的文件。

如果该文件存在，`helm package` 命令会在打包应用时忽略所有在`.helmignore`文件中匹配的文件。

这有助于避免不需要的或敏感文件及目录添加到你的helm chart中。

`.helmignore` 文件支持Unix shell的全局匹配，相对路径匹配，以及反向匹配（以！作为前缀）。每行只考虑一种模式。

这里是一个`.helmignore`文件示例：

```bash
# comment

# Match any file or path named .helmignore
.helmignore

# Match any file or path named .git
.git

# Match any text file
*.txt

# Match only directories named mydir
mydir/

# Match only text files in the top-level directory
/*.txt

# Match only the file foo.txt in the top-level directory
/foo.txt

# Match any file named ab.txt, ac.txt, or ad.txt
a[b-d].txt

# Match any file under subdir matching temp*
*/temp*

*/*/temp*
temp?
```

一些值得注意的和.gitignore不同之处：

- 不支持'**'语法。
- globbing库是Go的 'filepath.Match'，不是fnmatch(3)
- 末尾空格总会被忽略(不支持转义序列)
- 不支持'!'作为特殊的引导序列
- 默认不会排除自身，需要显式添加 `.helmignore`

# 十一、调试模板

调试模板可能很棘手，因为渲染后的模板发送给了Kubernetes API server，可能会以格式化以外的原因拒绝YAML文件。

以下命令有助于调试：

- `helm lint` 是验证chart是否遵循最佳实践的首选工具。
- `helm template --debug` 在本地测试渲染chart模板。
- `helm install --dry-run --debug`：我们已经看到过这个技巧了，这是让服务器渲染模板的好方法，然后返回生成的清单文件。
- `helm get manifest`: 这是查看安装在服务器上的模板的好方法。

当你的YAML文件解析失败，但你想知道生成了什么，检索YAML一个简单的方式是注释掉模板中有问题的部分， 然后重新运行 `helm install --dry-run --debug`：

```yaml
apiVersion: v2
# some: problem section
# {{ .Values.foo | quote }}
```

以上内容会被渲染同时返回完整的注释：

```yaml
apiVersion: v2
# some: problem section
#  "bar"
```

这样就提供了一种快速查看没有被YAML错误解析阻塞的生成内容的方式。



# 十二、helm高级操作



