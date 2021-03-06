# HTTP缓存

## 一、什么是缓存

Web缓存是可以自动保存常见文档副本的HTTP设备。当Web请求抵达缓存时，如果本地有“已缓存的”副本，就可以从本地存储设备，而不是原始服务器中提取这个文档。

## 二、缓存的优点

### 1. 减少了冗余的数据传输

很多客户端访问同一个流行的原始服务器页面时，服务器会多次传输同一份文档，每次传送给一个客户端。一些相同的字节会在网络中一遍遍的传输。这些冗余的数据传输会耗尽昂贵的网络带宽，降低传输速度，加重Web服务器的负载。

有了缓存，就可以保留第一条服务器响应的副本，后继请求就可以由缓存的副本来应对了，这样可以减少那些流入/流出原始服务器的、被浪费掉了的重复流量。

### 2. 缓解了带宽瓶颈

很多网络为本地网络客户端提供的带宽比为远程服务器提供的带宽要宽。客户端会以路径上最慢的网速访问服务器。如果客户端从一个快速局域网的缓存中得到了一份副本，那么缓存就可以提高性能，尤其是要传输比较大的文件时。

![图片](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b184ebeeed804879b134391572a86548~tplv-k3u1fbpfcp-watermark.image)

如上图所示，金山分店的用户通过1.4Mbit/s的T1因特网连接，从亚特兰大总店下载一个5MB的库存文件要花30秒的时间。如果在旧金山分店里缓存了这个文档，本地用户通过以太网连接只要花费不到1秒的时间就可以获得同一份文档了。

### 3. 破坏了瞬间拥塞

当发生突发事件，比如爆炸性新闻或者某个名人事件时，使得很多人几乎同一时间去访问一个Web文档时，就会出现瞬间拥塞。由此造成的过多流量峰值可能会使网络和Web服务器产生灾难性的崩溃。而缓存的存在可以破坏这种情况的发生。

### 4. 降低了距离时延

每台网络路由器都会增加网络流量的时延，即使客户端和服务器之间没有太多的路由器，光速自身也会造成显著的时延。

将缓存放在附近的机房里可以将文件传输距离从数千公里缩短为数十米。

## 三、命中的和未命中的

缓存无法保存世界上每份文档的副本。

可以用已有的副本为某些到达缓存的请求提供服务，这被称为`缓存命中`。见下图(a)。

其他一些到达缓存的请求可能会由于没有副本可用，而被转发给原始服务器，这被称为`缓存未命中`。见下图(b)。

![缓存命中、未命中以及再验证](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2bfc15f1c6a411aaff4def04938378a~tplv-k3u1fbpfcp-watermark.image)

原始服务器的内容可能会发生变化，缓存要不时地对其进行检测，看看它们保存的副本是否仍然是服务器上最新的副本。这些`“新鲜度检测”`被称为`HTTP再验证`。为了有效的进行再验证，HTTP定义了一些特殊的请求，不再从服务器上获取整个对象，就可以快速的检测出内容是最新的。

缓存可以在任意时刻，以任意的频率对副本进行再验证。但由于缓存中通常会包含数百万的文档，而且网络带宽是很珍贵的，所以大部分缓存只有在客户端发起请求，并且副本旧的足以需要检测的时候才会对副本进行再验证。

缓存对缓存的副本进行再验证时，会向原始服务器发送一个小的再验证请求。如果内容没有变化，服务器会以一个小的`304 Not Modified`进行响应。只要缓存知道副本仍然有效，就会再次将副本标识为暂时新鲜的，并将副本提供给客户端，这被称为`再验证命中`或`缓慢命中`。见下图(a)。这种方式确实要与原始服务器进行核对，所以会比单纯的缓存命中要慢，但它没有从服务器中获取对象数据，所以要比缓存未命中快一些。

![再验证](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/031a72bce4b143f6bedb454aa15c09b6~tplv-k3u1fbpfcp-watermark.image)

成功的再验证比缓存未命中要快，但是失败的再验证几乎和未命中的速度一样。

HTTP为我们提供了几个用来对已缓存对象进行再验证的工具，但最常用的`If-Modified-Since`首部。将这个首部添加到GET请求中去，就可以告诉服务器，只有在缓存了对象的副本之后，又对其进行了修改的情况下，才发送此对象。

这里列出了在3种情况下（服务器内容未被修改，服务器内容已被修改，或者服务器上的对象被删除了）服务器收到GET If-Modified-Since请求时会发生的情况：

（1）再验证命中：如果服务器对象未被修改，服务器会向客户端发送一个小的`304 Not Modified`进行响应。

（2）再验证未命中：如果服务器对象与已缓存副本不同，服务器向客户端发送一条普通的、带有完整内容的`HTTP 200 OK`响应

（3）对象被删除：如果服务器对象已经被删除了，服务器就回送一个`404 Not Found`响应，缓存也会将其副本删除。

## 四、缓存的处理步骤

现代的商业化代理缓存相当地复杂。这些缓存构建得非常高效，可以支持HTTP和其他一些技术的各种高级特性。但除了一些微妙的细节之外，Web缓存的基本工作原理大多很简单。对一条HTTP GET报文的基本缓存处理过程包括7个步骤，如下图所示。

![处理一个新鲜的缓存命中](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b8aa10fb3f764d5a921e743979a65c4b~tplv-k3u1fbpfcp-watermark.image)

### 第一步：接收

缓存检测到一条网络连接上的活动，从网络中读取抵达的请求报文。高性能的缓存会同时从多条输入连接上读取数据，在整条报文抵达之前开始对事务进行处理。

### 第二步：解析

缓存将请求报文解析为片断，提取出URL和各个首部，将首部的各个部分放入易于操作的数据结构中。这样，缓存软件就更容易处理首部字段并修改它们了。

### 第三步：查找

缓存已经获取了URL，开始查找是否有本地副本可用。如果没有，就获取一份副本，并将其保存在本地。

本地副本可能存储在内存、本地磁盘，甚至附近的另一台计算机中。专业级的缓存会使用快速算法来确定本地缓存中是否有某个对象。如果本地没有这个文档，它可以根据情形和配置，到原始服务器或父代理中去取，或者返回一条错误信息。

已缓存对象中包含了服务器响应主体和原始服务器响应首部，这样就会在缓存命中时返回正确的服务器首部。已缓存对象中还包含了一些元数据（metadata），用来记录对象在缓存中停留了多长时间以及它被用过多少次等。

### 第四步：新鲜度检测

HTTP通过缓存将服务器文档的副本保留一段时间。在这段时间里，都认为文档是“新鲜的”，缓存可以在不联系服务器的情况下，直接提供该文档。

但一旦已缓存副本停留的时间太长，超过了文档的新鲜度限值，就认为对象“过时”了，在提供该文档之前，缓存要再次与服务器进行确认，以查看文档是否发生了变化。

客户端发送给缓存的所有请求首部自身都可以强制缓存进行再验证，或者完全避免验证，这使得事情变得更复杂了。

### 第五步：创建响应

我们希望缓存的响应看起来就像来自原始服务器的一样，缓存将已缓存的服务器响应首部作为响应首部的起点。然后缓存对这些基础首部进行了修改和补充。

缓存负责对这些首部进行改造，以便与客户端的要求相匹配。比如，服务器返回的可能是一条HTTP/1.0响应（甚至是HTTP/0.9响应），而客户端期待的是一条HTTP/1.1响应，在这种情况下，缓存必须对首部进行相应的转换。缓存还会向其中插入新鲜度信息（Cache-Control、Age以及Expires首部），而且通常会包含一个Via首部来说明请求是由一个代理缓存提供的。

注意，缓存不应该调整Date首部。Date首部表示的是原始服务器最初产生这个对象的日期。

缓存用新的首部和已缓存的主体来构建一条响应报文。

### 第六步：发送

一旦响应首部准备好了，缓存就将响应回送给客户端。和所有的代理服务器一样，代理缓存要管理与客户端之间的连接。高性能的缓存会尽力高效的发送数据，通常可以避免在本地缓存和网络I/O缓冲区之间进行文档内容的复制。

### 第七步：日志

大多数缓存都会保存日志文件以及与缓存的使用有关的一些统计数据。每个缓存事务结束之后，缓存都会更新缓存命中和未命中数目的统计数据以及其他相关的度量值，并将条目插入一个用来显示请求类型、URL和所发生事件的日志文件。

下图以简化的形式显示了缓存是如何处理请求，以GET一个URL的。

![缓存GET请求的流程图](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e1f66f431244b8c85304556fcc79ece~tplv-k3u1fbpfcp-watermark.image)

## 五、保持副本的新鲜

可能不是所有的已缓存副本都与服务器上的文档一致。毕竟，这些文档会随着时间发生变化。报告可能每个月都会变化。在线报纸每天都会发生变化。财经数据可能每过几秒就会发生变化。如果缓存提供的总是老数据，就会变得毫无用处。已缓存数据要与服务器数据保持一致。

HTTP有一些简单的机制可以在不要求服务器记住哪些缓存拥有其文档副本的情况下，保持已缓存数据与服务器数据之间充分一致。

HTTP将这些简单的机制称为`文档过期`和`服务器再验证`。

### 1. 文档过期

通过特殊的HTTP `Cache-Control`首部和`Expires`首部，HTTP让原始服务器向每个文档加了一个“过期日期”，这些首部说明了在多长时间内可以将这些内容视为新鲜的。

在缓存文档过期之前，缓存可以以任意频率使用这些副本，而无需与服务器联系——当然，除非客户端请求中包含有阻止提供已缓存或未验证资源的首部。

但一旦已缓存文档过期，缓存就必须与服务器进行核对，询问文档是否被修改过，如果被修改过，就要获取一份新鲜（带有新的过期日期）的副本。

### 2. 过期日期与使用期

服务器用HTTP/1.0+的`Expires`首部或者HTTP/1.1的`Cache-Control:max-age`响应首部来指定过期日期，同时还会带有响应主体。

`Expires`首部和`Cache-Control:max-age`首部所做的事情本质上是一样的，但由于`Cache-Control`首部使用的是相对时间而不是绝对日期，所以我们**更倾向于使用比较新的`Cache-Control`首部**。而绝对日期依赖于计算机时钟的正确设置。

![过期响应首部](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b96ec3eddf284eaa8f3eeec5f676e0c0~tplv-k3u1fbpfcp-watermark.image)

### 3. 服务器再验证

仅仅是已缓存文档过期了并不意味着它和原始服务器上目前处于活跃状态的文档有实际的区别；这只是意味着到了要进行核对的时间了。这种情况被称为`“服务器再验证”`，说明`缓存需要询问原始服务器文档是否发生了变化`。

- 如果再验证显示内容发生了变化，缓存会获取一份新的文档副本，并将其存储在旧文档的位置上，然后将文档发送给客户端
- 如果再验证显示内容没有发生变化，缓存只需要获取新的首部，包括一个新的过期日期，并对缓存中的首部进行更新就行了

这是个很棒的系统，缓存并不一定要为每条请求验证文档的有效性——只有在文档过期时它才需要与服务器进行再验证。这样就不会提供陈旧的内容，还可以节省服务器的流量，并拥有更好的用户响应时间。

HTTP协议要求行为正确的缓存返回下列内容之一：

- “足够新鲜”的已缓存副本
- 与服务器进行再验证，确认其仍然新鲜的已缓存副本
- 如果需要与之进行再验证的原始服务器出故障了，就返回一条错误报文
- 附有警告信息说明内容可能不正确的已缓存副本

### 4. 用条件方法进行再验证

HTTP的条件方法可以高效的实现再验证。HTTP允许缓存向原始服务器发送一个“条件GET”，请求服务器只有在文档与缓存中现有的副本不同时，才回送对象主体。通过这种方式，将新鲜度检测和对象获取结合成了单个条件GET。向GET请求报文中添加一些特殊的条件首部，就可以发起条件GET。只有条件为真时，web服务器才会返回对象。

HTTP定义了5个条件请求首部。对缓存再验证来说最有用的2个首部是`If-Modified-Since`和`If-None-Match`。所有的条件首部都以前缀If-开头。还有三个是If-Unmodified-Since（在进行部分文件的传输时，获取文件的其余部分之前要确保文件未发生变化，此时这个首部是非常有用的）、If-Range（支持对不完整文档的缓存）和If-Match（用于与web服务器打交道时的并发控制）。

![缓存再验证中使用的两个条件首部](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5820430fb1094d3aaa4f3cc8e3c31396~tplv-k3u1fbpfcp-watermark.image)

### 5. If-Modified-Since:Date 再验证

这是最常见的缓存再验证首部。`If-Modified-Since`再验证请求通常被称为`IMS请求`。只有自某个日期之后资源发生了变化的时候，IMS请求才会指示服务器执行请求：

- 如果自指定日期后，文档被修改了，`If-Modified-Since`条件就为真，通常GET就会成功执行。携带新的首部的新文档会被返回给缓存，新首部出了其他信息之外，还包含了一个新的过期日期。
- 如果自指定日期后，文档没被修改过，条件就为假，会向客户端返回一个小的`304 Not Modified`响应报文，为了提高有效性，不会返回文档的主体。这些首部是放在响应中返回的，但只会返回那些需要在源端更新的首部。比如Content-Type首部通常不会被修改，所以通常不需要发送。一般会发送一个新的过期日期。

`If-Modified-Since`首部可以与`Last-Modified`服务器响应首部配合工作。原始服务器会将最后的修改日期附加到所提供的文档上去。当缓存要对已缓存文档进行再验证时，就会包含一个`If-Modified-Since`首部，最后携带有最后修改已缓存副本的日期：

```html
If-Modified-Since: <cached last-modified date>
```

如果在此期间，内容被修改了，最后的修改日期就会有所不同，原始服务器就会回送新的文档。否则，服务器会注意到缓存的最后修改日期与服务器文档当前的最后修改日期相符，会返回一个`304 Not Modified`响应

注意，有些web服务器并没有将If-Modified-Since作为真正的日期来进行比对，相反，它们在IMS日期和最后修改日期之间进行了字符串匹配。这样得到的语义就是“如果最后的修改不是在这个确定的日期进行的，而不是“如果在这个日期之后没有被修改过”。将最后修改日期作为某种序列号使用时，这种替代语义可以很好的识别出缓存是否过期，但这会妨碍客户端将If-Modified-Since首部用于真正基于时间的一些目的。

### 6. If-None-Match：实体标签再验证

有些情况仅仅使用最后修改日期进行再验证是不够的。

- 有些文档可能会被周期性的重写，比如从一个后台进程中写入，但实际包含的数据常常是一样的。尽管内容没有变化，但是修改日期会发生变化。
- 有些文档可能被修改了，但所做修改并不重要，不需要让世界范围内的缓存都重装数据，比如对拼写或者注释的修改
- 有些服务器无法准确的判定其页面的最后修改日期
- 有些服务器提供的文档会在亚秒间隙发生变化，比如实时监视器，对这些服务器来说，以一秒为粒度的修改日期可能就不够用了。

为了解决这些问题，HTTP允许用户对被称为`实体标签ETag`的“版本标识符”进行比较。`实体标签`是附加到文档上的任意标签（引用字符串）。她们可能包含了文档的序列号或版本名，或者是文档内容的校验和其他指纹信息。

当发布者对文档进行修改时，可以修改文档的`实体标签`来说明这是个新的版本。这样，如果`实体标签`被修改了，缓存就可以用`If-None-Match`条件首部来GET文档的新副本了。

可以在`If-None-Match`首部包含几个实体标签，告诉服务器，缓存中已经存在带有这些实体标签的对象副本。

![实体标签](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91ca7d38bc6445a19c3f1a2914e7f6b4~tplv-k3u1fbpfcp-watermark.image)

### 7. 强弱校验器

缓存可以用`实体标签`来判断，与服务器对比，已缓存版本是不是最新的（与使用最近修改日期的方式很像）。从这个角度来看，实体标签和最近修改日期都是缓存验证器。

有时，服务器希望在对文档进行一些非实质性或不重要的修改时，不要使所有的已缓存都失效。HTTP/1.1支持`“弱验证器”`，如果只对内容进行了少量修改，就允许服务器声明那是“足够好”的等价体。

只要内容发生了变化，`强验证器`就会变化。`弱验证器`允许对一些内容进行修改，但内容的主要含义发生变化时，通常它还是会变化的。有些操作不能用`弱验证器`来实现，比如有条件的获取部分内容，所以服务器会用前缀“W/”来标识弱验证器。

```html
ETag: W/"v2.6"
If-None-Match: W/"v2.6"
```

不管相关的实体值以何种方式发生了变化，`强实体标签`都要发生变化，而相关实体在语义上发生了比较重要的变化时，`弱实体标签`也应该发生变化。

### 8. 什么时候应该使用实体标签和最近修改日期

如果服务器回送了一个`实体标签`，HTTP/1.1客户端就必须使用`实体标签验证器`。如果服务器只回送了一个`Last-Modified`值，客户端就可以使用`If-Modified-Since`验证。如果`实体标签`和`Last-Modified`都提供了，客户端就应该使用这两种再验证方案，这样HTTP/1.0和HTTP/1.1就都可以正确响应了。

除非HTTP/1.1原始服务器无法生成实体标签验证器，否则就应该发送一个出去，如果使用弱实体标签有优势的话，发送的可能就是个弱实体标签，而不是强实体标签。而且，最好同时发送一个最近修改值。

如果HTTP/1.1缓存或服务器收到的请求既带有`If-Modified-Since`，又带有`实体标签`条件首部，那么**只有这两个条件都满足时，才能返回`304 Not Modified`**。

## 六、控制缓存的能力

服务器可以通过HTTP定义的几种方式来指定在文档过期之前可以将其缓存多长时间，按照优先级递减的顺序，服务器可以：

- 附加一个Cache-Control：no-store首部到响应中去
- 附加一个Cache-Control：no-cache首部到响应中去
- 附加一个Cache-Control：must-revalidate首部到响应中去
- 附加一个Expires首部到响应中去
- 不附加过期信息，让缓存确定自己的过期日期

### 1. no-store 和 no-cache 响应首部

HTTP/1.1提供了几种限制对象缓存，或限制提供已缓存对象的方式，以维持对象的新鲜度。

`no-store`首部和`no-cache`首部可以防止缓存提供未经证实的已缓存对象。

```html
Pragma: no-cache
Cache-Control: no-store
Cache-Control: no-cache
```

标识为`no-store的响应会禁止缓存对响应进行复制`。缓存通常会像非缓存代理服务器一样，向客户端转发一条no-store响应，然后删除对象

`标识为no-cache的响应实际上是可以存储在本地缓存区中的。`只是在与原始服务器进行新鲜度再验证之前，缓存不能将其提供给客户端使用。这个首部使用do-not-serve-from-cache-without-revalidation这个名字会更恰当一些。

HTTP/1.1提供`Pragma: no-cache首部是为了兼容于HTTP/1.0+`。除了与只理解`Pragma: no-cache`的HTTP/1.0应用程序进行交互时，HTTP 1.1应用程序都应该使用`Cache-Control：no-cache`

### 2. max-age 响应首部

Cache-Control：max-age首部表示的是从服务器将文档传来时起，可以认为此文档处于新鲜状态的秒数。还有一个`s-maxage`首部，其行为与`max-age`类似，但仅适用于共享（公有）缓存：

```html
Cache-Control: max-age=3600
Cache-Control: s-maxage=3600
```

服务器可以请求缓存不要缓存文档，或者将最大使用期设置为零，从而在每次访问的时候都进行刷新。

```html
Cache-Control: max-age=0
Cache-Control: s-maxage=0
```

### 3. Expires 响应首部

不推荐使用`Expires`首部，它指定的是`实际的过期日期而不是秒数`，HTTP设计者后来认为，由于很多服务器的时钟都不同步或者不正确，所以最好还是用剩余秒数，而不是绝对时间来表示过期时间。可以通过计算过期值和日期值之间的秒数差来计算类似的新鲜生存期：

```html
Expires: Fri, 05 Jul 2022, 05:00:00 GMT
```

### 4. must-revalidate 响应首部

可以配置缓存，使其提供一些陈旧过期的对象，以提高性能。如果原始服务器希望缓存严格遵守过期信息，可以在原始响应中附加一个`Cache-Control：must-revalidate`的首部。

这个响应首部告诉缓存，在`事先没有跟原始服务器进行再验证的情况下，不能提供这个对象的陈旧副本`。缓存仍然可以随意提供新鲜的副本。如果在缓存进行`must-revalidate`新鲜度检查时，原始服务器不可用，缓存就必须返回一条`504 Gateway Timeout`的错误。

### 5. 试探性过期

如果响应中没有`Cache-Control: max-age`首部，也没有`Expires`首部，缓存可以计算出一个试探性最大使用期。可以使用任意算法，但如果得到的最大使用期大于24小时，就应该向响应首部添加一个`Heuristic Expiration Warning`（试探性过期警告）首部，据我们所知，很少有浏览器会为用户提供这种警告信息。

`LM-Factor算法`是一种很常用的试探性过期算法。如果文档中包含了最后修改日期，就可以使用这种算法。`LM-Factor算法`将最后修改日期作为依据，来估计文档有多么易变。算法的逻辑如下所示：

- 如果已缓存文档最后一次修改发生在很久以前，它可能会是一份稳定的文档，不太会突然发生变化，因此将其继续保存在缓存中会比较安全。
- 如果已缓存文挡最近被修改过，就说明它很可能会频繁地发生变化，因此在与服务器进行再验证之前，只应该将其缓存很短一段时间。

实际的`LM-Factor算法`会计算缓存与服务器对话的时间跟服务器声明文档最后被修改的时间之间的差值，取这个间隔时间的一部分，将其作为缓存中的新鲜度持续时间。

通常人们会为试探性新鲜周期设置上限，这样它们就不会变得太大了。尽管比较保守的站点会将这个值设置为一天，但通常站点会将其设置为一周。

如果最后修改日期也没有的话，缓存就没什么信息可利用了。缓存通常会为没有任何新鲜周期线索的文档分配一个默认的新鲜周期（通常是一个小时或一天）。有时，比较保守的缓存会将这种试探性新鲜生存期设置为0，强制缓存在每次将其提供给客户端之前，都去验证一下这些数据仍然是新鲜的。

与试探性新鲜计算有关的最后一点是——它们可能比你想象的要常见得多。很多原始服务器仍然不会产生`Expires`和`max-age`首部。选择缓存过期的默认时间时要特别小心！

### 6. 客户端的新鲜度限制

Web浏览器都有Refresh（刷新）或Reload（重载）按钮，可以强制对浏览器或代理缓存中可能过期的内容进行刷新。

Refresh按钮会发布一个附加了`Cache-Control`请求首部的GET请求，这个请求会`强制进行再验证，或者无条件地从服务器获取文档`。Refresh的确切行为取决于特定的浏览器、文档以及拦截缓存的配置。

![cache-control](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a93629cf06444adf86dbaea39b6f2c4a~tplv-k3u1fbpfcp-watermark.image)

## 七、缓存的分类

### 1. 按使用缓存的对象分类

缓存可以是单个用户专用的，也可以是数千名用户共享的。专用缓存被称为私有缓存。共享的缓存被称为公有缓存。

![公有和私有缓存](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/792ca823167840d381be749ad684c30b~tplv-k3u1fbpfcp-watermark.image)

#### 私有缓存

私有缓存不需要很大的动力或存储空间，这样就可以将其做的很小，很便宜。Web浏览器中有内建的私有缓存——大多数浏览器都会将常用文档缓存在你个人电脑的磁盘和内存中，并且允许用户去配置缓存的大小和各种设置。还可以去看看浏览器的缓存中有些什么内容。

#### 公有缓存

公有缓存是特殊的共享代理服务器，被称为缓存代理服务器，或者更常见地被称为代理缓存。代理缓存会从本地缓存中提供文档，或者代表用户与服务器进行联系。

公有缓存会接受来自多个用户的访问，所以通过它可以更好的减少冗余流量。公有缓存要缓存用户群体中各种不同的兴趣点，所以要足够大才能承载常用的文档集，而不会被单个用户所感兴趣的文档占满。

如下图所示，每个客户端都会重复访问一个新的“热门”文档。每个私有缓存都要获取同一份文档，这样它就会多次穿过网络。而使用共享的公有缓存时，对于这个流行的对象，缓存只要取一次就行了，它会用共享的副本为所有的请求服务，以降低网络流量。

![共享的公有缓存可以降低网络流量](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/454833b3b12a481c8fccb5d761e7fc58~tplv-k3u1fbpfcp-watermark.image)

### 2. 按缓存位置分类

- Service Worker
- Memory Cache
- Disk Cache

浏览器按照从上往下的顺序逐层查找，找到要请求的内容就返回，否则就向服务器发起请求。

#### Memory Cache

`memory cache`是内存中的缓存

几乎所有的网络请求资源都会被浏览器自动加入到`memory cache`中，但是也正因为数量很大而浏览器占用的内存不能无限扩大这两个因素，`memory cache`就注定只能是个`“短期存储”`。

常规情况下，浏览器的Tab关闭后该浏览器的memory cache就会失效，而极端情况下，比如一个页面的缓存占用了超级多的内存，那么可能在Tab没关闭之前，排在前面的缓存就已经失效了。

前面讲到，几乎所有的请求资源都能进入memory cache，这里细分一下主要有两块：

1. preloader。熟悉浏览器渲染流程的同学们应该了解，浏览器会先请求HTML然后解析，而JavaScript、CSS文件的下载和执行都会阻塞HTML的解析，所以解析到HTML中包含相关的JavaScript、CSS文件时，可以提前下载这些文件，这就是preloader。这些下载好的资源就会被放入到memory cache中
2. preload。例如`<link rel="preload">`，显示指定的预加载资源，也会被放入memory cache中

`memory cache`保证了一个页面中如果有两个相同的请求，例如两个`src`相同的`<img>`，两个`href`相同的`<link>`时，实际都只会被请求最多一次。

在匹配缓存时，除了匹配完全相同的URL之外，还会比对它们的类型，CORS中的域名规则等。因此一个作为脚本类型被缓存的资源时不能用在图片类型的请求中的，即便它们的src相等。

从memory cache获取缓存内容时，浏览器会忽视如`max-age=0`，`no-cache`等头部配置。例如页面上存在几个相同的`src`图片，即便它们可能被设置为不缓存，但依然会从memory cache中读取。这是因为memory cache只是短期使用，大部分情况生命周期只有一次浏览而已。而`max-age=0`在语义上普遍被解读为“不要在下次浏览时使用”，所以和memory cache并不冲突。

但是如果不想资源进入缓存，甚至memory cache也不行，就需要使用`no-store`。

#### Disk Cache

也叫HTTP Cache。顾名思义，硬盘上的缓存，因此它是`持久存储的，实际存在于文件系统中的`。而且它允许相同的资源在跨会话，甚至跨站点的情况下使用。例如两个站点都使用了同一种图片。

`disk cache`会严格根据HTTP头部信息中的各类字段来判定哪些资源可以缓存，哪些资源不可以缓存；哪些资源是仍然可用的，哪些资源是过时需要重新请求的。当命中缓存之后，浏览器会从硬盘中读取资源，虽然比起从内存中读取慢了一些，但比起通过网络请求还是快了不少的。绝大部分的缓存都来自于`disk cache`。

理论上讲，当一个资源被缓存存储以后，该资源应该可以被永久的存储在缓存中。但是由于缓存只有有限的空间存储资源副本，所以缓存会定期的将一些副本删除，这个过程叫做**缓存驱逐**。

凡是持久性存储都会面临容量增长的问题。在浏览器自动清理时，会有神秘的算法把“最老的”、“最可能过时的”资源删除，因此时一个一个删除的。不过每个浏览器识别“最老的”和“最可能过时的”资源的算法不尽相同。

#### Service Worker

上述的缓存策略以及缓存/读取/失效的动作都是由浏览器内部判断和进行的。我们只能设置响应头的某些字段来告诉浏览器，而不能自己操作。就像去银行存取钱，你只能告诉银行职员，我要存/取多少钱，然后他们会经过一系列的记录和手续之后，把钱放到金库中/取出来给你。

但是`Service Worker`的出现给予了我们一种更加灵活、更加直接的操作方式。依然用存/取钱的例子，我们现在可以绕开银行职员，自己走到金库前把钱放进去或者取出来。当然这个金库不是上面那个金库，而是一个单独的金库。因此我们可以选择放哪些钱（缓存哪些文件），什么情况下把钱取出来（路由匹配规则），取哪些钱出来（缓存匹配并返回）。当然现实中银行没有给我们开放这样的服务。

`Service Worker`能够操作的缓存是有别于浏览器内部的`memory cache`和`disk cache`的。我们可以从Chrome的控制台，Application -> Cache Storage中找到这个小金库。除了位置不同以外，这个缓存是永久性的，即使关闭Tab或者浏览器，下次打开依然还在，而memory cache不是。

有两种情况会导致这个缓存中的资源被清除。手动调用API `cache.delete(resourse)`或者容量超过限制，被浏览器全部清空。

如果`Service Worker`没能命中缓存，一般情况下会使用`fetch()`继续获取资源，这时候浏览器就会去`memory cache`或者`disk cache`进行下一次的缓存匹配工作了。

注意：经过`Service Worker`的`fetch()`方法获取的资源，即便它没有命中`Service Worker`的缓存甚至直接走向了网络请求，也会被标注为`from ServiceWorker`。

#### 小结

当浏览器要请求资源时

1. 调用Service Worker的fetch事件响应
2. 查看memory cache
3. 查看disk cache，这里又可以细分为两种情况
    - 如果有强制缓存且未失效，则使用强制缓存，不请求服务器，状态码是200
    - 如果有强制缓存但已失效，使用对比缓存，比较后确定 304 还是 200
4. 发送网络请求，等待网络响应
5. 把响应内容存入disk cache（如果HTTP头信息配置可存的话）
6. 把响应内容的引用存入memory cache（无视HTTP头信息的配置）
7. 把响应内容存入Service Worker的Cache Storage（如果Service Worker的脚本调用了cache.put()）

### 3. 按失效策略分类

memory cache是浏览器为了加快读取缓存速度而进行的自身的优化行为，不受开发者控制，也不受HTTP协议头的约束，算是一个黑盒。service worker是由开发者编写的额外的脚本，且缓存位置独立，出现比较晚，使用还不算太广泛。所以我们平时最为熟悉的其实是disk cache，也就是HTTP cache。它遵守HTTP协议头中的字段。平时所说的强缓存、对比缓存以及`Cache-Control`等，也都归于此类。

![流程图](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d554fdbe1d7f4d6685f90d5ac0338d79~tplv-k3u1fbpfcp-watermark.image)

#### 强缓存

强缓存的含义是，当客户端请求后，会先访问缓存数据库看缓存是否存在，如果存在则直接返回，如果不存在，则请求真的服务器，响应后再写入缓存数据库。

强制缓存直接减少了请求数，是提升最大的缓存策略。如果考虑使用缓存来优化网页性能的话，强缓存应该是首先被考虑的。

可以造成强缓存的字段是`Cache-Control`、`Expires`和`Pragma`

#### 协商缓存

当强制缓存失效（超过规定时间）时，需要使用对比缓存，由服务器决定缓存内容是否失效。

流程上说，浏览器先请求缓存数据库，返回一个缓存标识，之后浏览器拿这个缓存标识和服务器通讯，如果缓存未失效，则返回HTTP状态码304表示继续使用，于是客户端继续使用缓存，如果失效，则返回新的数据和缓存规则，浏览器响应数据后，再把规则写入到数据库中

对比缓存在请求数层面上和没有缓存是一样的，但如果是304的话，返回的仅仅是一个状态码而已，并没有实际的文件内容，因此在响应体积上有节省的。通过减少响应体的体积来缩短网络传输的时间。所以和强制缓存相比较，性能的提升幅度较小，但是总比没有缓存好。

对比缓存是可以和强制缓存一起使用的，作为在强制缓存失效后的一种后备方案，实际项目中它们也的确经常一同出现

对比缓存相关的两组字段是：last-modified & If-modified-since、Etag & If-None-Match

##### Last-Modified 和 If-Modified-Since

二者的值都是`GMT`格式的时间字符串。

`Last-Modified`表示本地文件最后的修改时间，下一次请求时，浏览器会在请求头中会带上`If-Modified-Since`，它的值就是上次返回的`Last-Modified`，询问服务器在该日期后资源是否有更新。

如果文件没有变化，服务器则返回`304 Not Modified`的响应，不会返回资源内容，浏览器直接使用本地缓存。而且响应头中也不会再添加`Last-Modified`去试图更新本地缓存的`Last-Modified`，因为既然资源没有变化，那么`Last-Modifed`也就不会改变。

如果资源有变化，就正常返回资源的内容，新的`Last-Modified`也会在响应头中返回，并在下次请求之前更新本地缓存的`Last-Modified`，下次请求时，请求头中的`If-Modified-Since`会启用更新后的`Last-Modified`

但是如果在本地打开了缓存文件，会造成`Last-Modified`被修改，所以 HTTP/1.1 出现了`ETag`

##### ETag 和 If-None-Match

二者的值都是由服务器为每一个资源生成的唯一标识字符串，是一串哈希码，只要资源有变化，这个值就会改变。`ETag`可以保证每个资源都是唯一的。

请求资源时，服务器根据文件本身计算出一个哈希值，通过`ETag`字段返回给浏览器，当再一次请求时，在请求头中加上`If-None-Match`，值为上次返回的`ETag`，服务器接收到之后会通过比较资源的`ETag`和`If-None-Match`是否一致来判断文件内容是否被改变

与`Last-Modified`不同的是，当服务器返回`304 Not Modified`的响应时，由于在服务器上`ETag`重新计算了，所以响应头中还是会返回这个重新计算的`ETag`，即使这个`ETag`跟之前的没有变化。

`Last-Modified`和`ETag`是可以一起使用的，服务器会优先验证`ETag`，一致的情况下才会继续比对`Last-Modified`，最后才决定是否返回304.

##### 为什么要有ETag

`HTTP/1.1`中`ETag`的出现主要是为了解决几个`Last-Modified`比较难解决的问题：

- 一些文件也许会周期性的改变，但是内容并不改变，改变的仅仅是修改时间，这个时候我们并不希望客户端认为这个文件被修改了，而让服务器重新返回资源内容
- 某些文件修改非常频繁，比如在秒级以下的时间内进行修改，`If-Modified-Since`能检查到的粒度是秒级别的，使用`ETag`能够保证这种需求下客户端在1秒内能刷新N次cache
- 某些服务器不能精确的得到文件的最后修改时间

#### 优先级

强缓存 > 协商缓存

`Cache-Control` > `Expires` > `Etag` > `Last-Modified`

## 八、缓存策略的选择

对于频繁变动的资源，首先需要使用`Cache-Control: no-cache`使浏览器每次都请求服务器，然后配合 `Etag` 或者 `Last-Modified` 来验证资源是否有效。这样的做法虽然不能节省请求数量，但是能显著减少响应数据大小。

对于不常变化的资源，给它们的`Cache-Control`配置一个很大的`max-age=31536000` (一年)，这样浏览器之后请求相同的 URL 会命中强制缓存。而为了解决更新的问题，就需要在文件名（或者路径）中添加 hash， 版本号等动态字符，之后更改动态字符，从而达到更改引用 URL 的目的，让之前的强制缓存失效（其实并未立即失效，只是不再使用了而已）。在线提供的类库（如 jquery-3.3.1.min.js, lodash.min.js 等）均采用这个模式。

## 九、参考文章

- [一文读懂前端缓存](https://zhuanlan.zhihu.com/p/44789005)
- [缓存（二）——浏览器缓存机制：强缓存、协商缓存](https://github.com/amandakelake/blog/issues/41)
- 《HTTP权威指南》
