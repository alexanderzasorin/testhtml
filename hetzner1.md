Устанавливаем учебный кластер на Hetzner
===================

Регистрация
-------------

Тут инфа про регистрацию

Первая установка ubuntu, и ключей
-------------

#### На входе 

	логин: #427600+j\*\*\*\*\*\* 
	пароль \*\*\*\*\*\* 
	адрес: https://robot.your-server.de
#### Задача (для каждого сервера)

- узнать ip ноды
- прописать ключи для доступа к серверу
- установить ubuntu на ноде
	- сделать одну ноду для понадежнее (RAID 1), будет использоваться как namenode
	- сделать остальные ноды побыстрее (RAID 0): HDFS все равно займется репликацией

#### Решение
1. У себя на Макбуке в консоли генерируем private key, вводя пустую passphrase
```bash
$ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/topsolver/.ssh/id_rsa):bigdata.pem
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in bigdata.pem.
Your public key has been saved in bigdata.pem.pub.
...

```

Был сгенерирован заодно и public key, но если вдруг нужно получить публичный ключ по **уже имеющемуся** приватному, то

```bash
ssh-keygen -y -f bigdata.pem
ssh-rsa AAAAB...
...............
...pt0d36/CcZ
```
2. Логинимся на https://robot.your-server.de, и попадаем в консоль, во вкладке servers видим: ip-адрес сервера
![enter image description here](https://drive.google.com/file/d/0B4VcKk8tESBOYlBmZE5JV2xMSkk/view?usp=sharing)
2. В *Key management* -> *New key* добавляем public key 
![enter image description here](https://drive.google.com/file/d/0B4VcKk8tESBOYlBmZE5JV2xMSkk/view?usp=sharing)
3. В Servers -> Rescue выделяем Linux, 64 bit, все ключи
![enter image description here](https://drive.google.com/file/d/0B4VcKk8tESBOYlBmZE5JV2xMSkk/view?usp=sharing)
4. В Servers -> Reset делаем hard reset
![enter image description here](https://drive.google.com/file/d/0B4VcKk8tESBOYlBmZE5JV2xMSkk/view?usp=sharing)
5. В консоли подключаемся по ssh ```ssh -i bigdata.pem root@5.9.47.146```
7. Устанавливаем образ ubuntu
```bash
installimage -r yes -l 1 -b grub -n npl-hz-0-node1 -p "swap:swap:16G,/boot:ext3:512M,/:ext4:all" -d sda,sdb -s en -i root/.oldroot/nfs/images/Ubuntu-1404-trusty-64-minimal.tar.gz -K /root/.ssh/authorized_keys -a
```
> **installimage** - это специальная утилита hetzner. Толкового описания параметров у нее нет, но есть *installimage -h* и исходные коды, из которых можно понять, как работают параметры

| Параметр     | Что делает|
| :------- | :---- |
| ```-r yes``` | включаем RAID |
| ```-l 1```    | тип RAID (для namenode мы хотим зеркальный RAID 1)   |
| ```-b grub```     | тип загрузчика - просят не менять    |
| ```-n npl-hz-0-node1```     | имя хоста (внутреннее)    |
| ```-p "swap:swap:16G,/boot:ext3:512M,/:ext4:all"```     | partitions: все (кроме небольшого куска) место в одну партицию    |
| ```-d sda,sdb```     | используем оба жестких диска    |
| ```-s en```     | лучше английский, чем немецкий    |
| ```-i root/.oldroot/nfs/images/Ubuntu-1404-trusty-64-minimal.tar.gz```     | образ стабильной Ubuntu 1404    |
| ```-K /root/.ssh/authorized_keys```     | ключи копируем те же: bigdata    |
| ```-a```     | тихая установка в автоматическом режиме. Без этого ключа можно будет поменять вручную некоторые параметры перед запуском установки    |
Больше информации про hetzner installimage [тут](http://wiki.hetzner.de/index.php/Installimage/ru)
8. Подключаемся по ssh к ноде
```bash
#предварительно очищаем в файле authorized_hosts старый fingerprint (который остался сохранен на локальной машине с момента подключения к rescue linux, до установки нашего образа Ubuntu 14.04)
ssh-keygen -R 5.9.47.146
#теперь спокойно подключимся - приватный ключ подойдет
ssh -i bigdata.pem root@5.9.47.146
```
9. Повторяем для остальных нод, поменяв тип RAID на 0




#### Немного автоматизации от ansible
Нужно сделать какие-то действия для нескольких серверов. Чего бы не автоматизировать?

#### Часть действий все равно вручную
- Залогиниться в панель управления каждым сервером
- прописать ключи для rescue 
- запустить rescue
- сделать automated hard reset

#### Задача
- написать скрипт, которому можно скормить много ip-адресов и будущих названий хостов, ключ доступа - и он на все установит Ubuntu 14.04
	- учесть, что на ноде-1 нужно сделать RAID1, а на остальных RAID0
- запустить скрипт


df


ы
ы
ы
ы
ы
ы
ыы

ыы






Hey! I'm your first Markdown document in **StackEdit**[^stackedit]. Don't delete me, I'm very helpful! I can be recovered anyway in the **Utils** tab of the <i class="icon-cog"></i> **Settings** dialog.

----------


Documents
-------------

StackEdit stores your documents in your browser, which means all your documents are automatically saved locally and are accessible **offline!**

> **Note:**

> - StackEdit is accessible offline after the application has been loaded for the first time.
> - Your local documents are not shared between different browsers or computers.
> - Clearing your browser's data may **delete all your local documents!** Make sure your documents are synchronized with **Google Drive** or **Dropbox** (check out the [<i class="icon-refresh"></i> Synchronization](#synchronization) section).

#### <i class="icon-file"></i> Create a document

The document panel is accessible using the <i class="icon-folder-open"></i> button in the navigation bar. You can create a new document by clicking <i class="icon-file"></i> **New document** in the document panel.

#### <i class="icon-folder-open"></i> Switch to another document

All your local documents are listed in the document panel. You can switch from one to another by clicking a document in the list or you can toggle documents using <kbd>Ctrl+[</kbd> and <kbd>Ctrl+]</kbd>.

#### <i class="icon-pencil"></i> Rename a document

You can rename the current document by clicking the document title in the navigation bar.

#### <i class="icon-trash"></i> Delete a document

You can delete the current document by clicking <i class="icon-trash"></i> **Delete document** in the document panel.

#### <i class="icon-hdd"></i> Export a document

You can save the current document to a file by clicking <i class="icon-hdd"></i> **Export to disk** from the <i class="icon-provider-stackedit"></i> menu panel.

> **Tip:** Check out the [<i class="icon-upload"></i> Publish a document](#publish-a-document) section for a description of the different output formats.


----------


Synchronization
-------------------

StackEdit can be combined with <i class="icon-provider-gdrive"></i> **Google Drive** and <i class="icon-provider-dropbox"></i> **Dropbox** to have your documents saved in the *Cloud*. The synchronization mechanism takes care of uploading your modifications or downloading the latest version of your documents.

> **Note:**

> - Full access to **Google Drive** or **Dropbox** is required to be able to import any document in StackEdit. Permission restrictions can be configured in the settings.
> - Imported documents are downloaded in your browser and are not transmitted to a server.
> - If you experience problems saving your documents on Google Drive, check and optionally disable browser extensions, such as Disconnect.

#### <i class="icon-refresh"></i> Open a document

You can open a document from <i class="icon-provider-gdrive"></i> **Google Drive** or the <i class="icon-provider-dropbox"></i> **Dropbox** by opening the <i class="icon-refresh"></i> **Synchronize** sub-menu and by clicking **Open from...**. Once opened, any modification in your document will be automatically synchronized with the file in your **Google Drive** / **Dropbox** account.

#### <i class="icon-refresh"></i> Save a document

You can save any document by opening the <i class="icon-refresh"></i> **Synchronize** sub-menu and by clicking **Save on...**. Even if your document is already synchronized with **Google Drive** or **Dropbox**, you can export it to a another location. StackEdit can synchronize one document with multiple locations and accounts.

#### <i class="icon-refresh"></i> Synchronize a document

Once your document is linked to a <i class="icon-provider-gdrive"></i> **Google Drive** or a <i class="icon-provider-dropbox"></i> **Dropbox** file, StackEdit will periodically (every 3 minutes) synchronize it by downloading/uploading any modification. A merge will be performed if necessary and conflicts will be detected.

If you just have modified your document and you want to force the synchronization, click the <i class="icon-refresh"></i> button in the navigation bar.

> **Note:** The <i class="icon-refresh"></i> button is disabled when you have no document to synchronize.

#### <i class="icon-refresh"></i> Manage document synchronization

Since one document can be synchronized with multiple locations, you can list and manage synchronized locations by clicking <i class="icon-refresh"></i> **Manage synchronization** in the <i class="icon-refresh"></i> **Synchronize** sub-menu. This will let you remove synchronization locations that are associated to your document.

> **Note:** If you delete the file from **Google Drive** or from **Dropbox**, the document will no longer be synchronized with that location.

----------


Publication
-------------

Once you are happy with your document, you can publish it on different websites directly from StackEdit. As for now, StackEdit can publish on **Blogger**, **Dropbox**, **Gist**, **GitHub**, **Google Drive**, **Tumblr**, **WordPress** and on any SSH server.

#### <i class="icon-upload"></i> Publish a document

You can publish your document by opening the <i class="icon-upload"></i> **Publish** sub-menu and by choosing a website. In the dialog box, you can choose the publication format:

- Markdown, to publish the Markdown text on a website that can interpret it (**GitHub** for instance),
- HTML, to publish the document converted into HTML (on a blog for example),
- Template, to have a full control of the output.

> **Note:** The default template is a simple webpage wrapping your document in HTML format. You can customize it in the **Advanced** tab of the <i class="icon-cog"></i> **Settings** dialog.

#### <i class="icon-upload"></i> Update a publication

After publishing, StackEdit will keep your document linked to that publication which makes it easy for you to update it. Once you have modified your document and you want to update your publication, click on the <i class="icon-upload"></i> button in the navigation bar.

> **Note:** The <i class="icon-upload"></i> button is disabled when your document has not been published yet.

#### <i class="icon-upload"></i> Manage document publication

Since one document can be published on multiple locations, you can list and manage publish locations by clicking <i class="icon-upload"></i> **Manage publication** in the <i class="icon-provider-stackedit"></i> menu panel. This will let you remove publication locations that are associated to your document.

> **Note:** If the file has been removed from the website or the blog, the document will no longer be published on that location.

----------


Markdown Extra
--------------------

StackEdit supports **Markdown Extra**, which extends **Markdown** syntax with some nice features.

> **Tip:** You can disable any **Markdown Extra** feature in the **Extensions** tab of the <i class="icon-cog"></i> **Settings** dialog.

> **Note:** You can find more information about **Markdown** syntax [here][2] and **Markdown Extra** extension [here][3].


### Tables

**Markdown Extra** has a special syntax for tables:

Item     | Value
-------- | ---
Computer | $1600
Phone    | $12
Pipe     | $1

You can specify column alignment with one or two colons:

| Item     | Value | Qty   |
| :------- | ----: | :---: |
| Computer | $1600 |  5    |
| Phone    | $12   |  12   |
| Pipe     | $1    |  234  |


### Definition Lists

**Markdown Extra** has a special syntax for definition lists too:

Term 1
Term 2
:   Definition A
:   Definition B

Term 3

:   Definition C

:   Definition D

	> part of definition D


### Fenced code blocks

GitHub's fenced code blocks are also supported with **Highlight.js** syntax highlighting:

```
// Foo
var bar = 0;
```

> **Tip:** To use **Prettify** instead of **Highlight.js**, just configure the **Markdown Extra** extension in the <i class="icon-cog"></i> **Settings** dialog.

> **Note:** You can find more information:

> - about **Prettify** syntax highlighting [here][5],
> - about **Highlight.js** syntax highlighting [here][6].


### Footnotes

You can create footnotes like this[^footnote].

  [^footnote]: Here is the *text* of the **footnote**.


### SmartyPants

SmartyPants converts ASCII punctuation characters into "smart" typographic punctuation HTML entities. For example:

|                  | ASCII                        | HTML              |
 ----------------- | ---------------------------- | ------------------
| Single backticks | `'Isn't this fun?'`            | 'Isn't this fun?' |
| Quotes           | `"Isn't this fun?"`            | "Isn't this fun?" |
| Dashes           | `-- is en-dash, --- is em-dash` | -- is en-dash, --- is em-dash |


### Table of contents

You can insert a table of contents using the marker `[TOC]`:

[TOC]


### MathJax

You can render *LaTeX* mathematical expressions using **MathJax**, as on [math.stackexchange.com][1]:

The *Gamma function* satisfying $\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$ is via the Euler integral

$$
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.
$$

> **Tip:** To make sure mathematical expressions are rendered properly on your website, include **MathJax** into your template:

```
<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML"></script>
```

> **Note:** You can find more information about **LaTeX** mathematical expressions [here][4].


### UML diagrams

You can also render sequence diagrams like this:

```sequence
Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob-->Alice: I am good thanks!
```

And flow charts like this:

```flow
st=>start: Start
e=>end
op=>operation: My Operation
cond=>condition: Yes or No?

st->op->cond
cond(yes)->e
cond(no)->op
```

> **Note:** You can find more information:

> - about **Sequence diagrams** syntax [here][7],
> - about **Flow charts** syntax [here][8].

### Support StackEdit

[![](https://cdn.monetizejs.com/resources/button-32.png)](https://monetizejs.com/authorize?client_id=ESTHdCYOi18iLhhO&summary=true)

  [^stackedit]: [StackEdit](https://stackedit.io/) is a full-featured, open-source Markdown editor based on PageDown, the Markdown library used by Stack Overflow and the other Stack Exchange sites.


  [1]: http://math.stackexchange.com/
  [2]: http://daringfireball.net/projects/markdown/syntax "Markdown"
  [3]: https://github.com/jmcmanus/pagedown-extra "Pagedown Extra"
  [4]: http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference
  [5]: https://code.google.com/p/google-code-prettify/
  [6]: http://highlightjs.org/
  [7]: http://bramp.github.io/js-sequence-diagrams/
  [8]: http://adrai.github.io/flowchart.js/
