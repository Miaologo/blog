---
title: 如何用python发邮件
date: 2020-05-11 23:43:48
tags:
---

## 用Python发送邮件，需要这样三步

> [用Python发送邮件，需要这样三步](https://www.epubit.com/articleDetails?id=11c1d7eaa14e4f0dbfac84e307ed166d)

检查和答复电子邮件会占用大量的时间。当然，你不能只写一个程序来处理所有电子邮件，因为每个消息都需要有自己的回应。但是，一旦知道怎么编写收发电子邮件的程序，就可以自动化大量与电子邮件相关的任务。

例如，也许你有一个电子表格，包含许多客户记录，希望根据他们的年龄和位置信息，向每个客户发送不同格式的邮件。商业软件可能无法做这一点。好在，可以编写自己的程序来发送这些电子邮件，节省了大量复制和粘贴电子邮件的时间。

也可以编程发送电子邮件和短信，即使你远离计算机时，也能通知你。如果要自动化的任务需要执行几个小时，你不希望每过几分钟就回到计算机旁边，检查程序的状态。相反，程序可以在完成时向手机发短信，让你在离开计算机时，能专注于更重要的事情。

## 16.1 SMTP

正如HTTP是计算机用来通过因特网发送网页的协议，简单邮件传输协议（SMTP）是用于发送电子邮件的协议。SMTP 规定电子邮件应该如何格式化、加密、在邮件服务器之间传递，以及在你点击发送后，计算机要处理的所有其他细节。但是，你并不需要知道这些技术细节，因为Python的smtplib模块将它们简化成几个函数。

SMTP只负责向别人发送电子邮件。另一个协议，名为IMAP，负责取回发送给你的电子邮件，在16.3节“IMAP”中介绍。

## 16.2 发送电子邮件

你可能对发送电子邮件很熟悉，通过Outlook、Thunderbird或某个网站，如Gmail或雅虎邮箱。遗憾的是，Python没有像这些服务一样提供一个漂亮的图形用户界面。作为替代，你调用函数来执行SMTP的每个重要步骤，就像下面的交互式环境的例子。

> {注意}
>
> 
>
> 不要在IDLE中输入这个例子，因为smtp.example.com、
>
> bob@example.com
>
> 、MY_ SECRET_PASSWORD和
>
> alice@example.com
>
> 只是占位符。这段代码仅仅勾勒出Python发送电子邮件的过程。

```
>>> import smtplib>>> smtpObj = smtplib.SMTP('smtp.example.com', 587)>>> smtpObj.ehlo()(250, b'mx.example.com at your service, [216.172.148.131]\nSIZE 35882577\n8BITMIME\nSTARTTLS\nENHANCEDSTATUSCODES\nCHUNKING')>>> smtpObj.starttls()(220, b'2.0.0 Ready to start TLS')>>> smtpObj.login('bob@example.com', 'MY_SECRET_PASSWORD')(235, b'2.7.0 Accepted')>>> smtpObj.sendmail('bob@example.com', 'alice@example.com', 'Subject: Solong.\nDear Alice, so long and thanks for all the fish. Sincerely, Bob'){}>>> smtpObj.quit()(221, b'2.0.0 closing connection ko10sm23097611pbd.52 - gsmtp')
```

在下面的小节中，我们将探讨每一步，用你的信息替换占位符，连接并登录到SMTP服务器，发送电子邮件，并从服务器断开连接。

### 16.2.1 连接到SMTP服务器

如果你曾设置了Thunderbird、Outlook或其他程序，连接到你的电子邮件账户，你可能熟悉配置SMTP服务器和端口。这些设置因电子邮件提供商而不同，但在网上搜索“< 你的提供商> SMTP设置”，应该能找到相应的服务器和端口。

SMTP服务器的域名通常是电子邮件提供商的域名，前面加上SMTP。例如，Gmail的 SMTP 服务器是smtp.gmail.com。表 16-1 列出了一些常见的电子邮件提供商及其SMTP服务器（端口是一个整数值，几乎总是587，该端口由命令加密标准TLS使用）。

表16-1 电子邮件提供商及其SMTP服务器

| 提供商                  | SMTP服务器域名                 |
| ----------------------- | ------------------------------ |
| Gmail                   | *smtp.gmail.com*               |
| Outlook.com/Hotmail.com | *smtp-mail.outlook.com*        |
| Yahoo Mail              | *smtp.mail.yahoo.com*          |
| AT&T                    | *smpt.mail.att.net* (port 465) |
| Comcast                 | *smtp.comcast.net*             |
| Verizon                 | *smtp.verizon.net* (port 465)  |

得到电子邮件提供商的域名和端口信息后，调用smtplib.SMTP()创建一个SMTP对象，传入域名作为一个字符串参数，传入端口作为整数参数。SMTP对象表示与SMTP邮件服务器的连接，它有一些发送电子邮件的方法。例如，下面的调用创建了一个SMTP对象，连接到Gmail：

```
>>> smtpObj = smtplib.SMTP('smtp.gmail.com', 587)>>> type(smtpObj)< class 'smtplib.SMTP'>
```

输入type(smtpObj)表明，smtpObj中保存了一个SMTP对象。你需要这个SMTP对象，以便调用它的方法，登录并发送电子邮件。如果smtplib.SMTP()调用不成功，你的SMTP服务器可能不支持TLS端口587。在这种情况下，你需要利用smtplib.SMTP_SSL()和465端口，来创建SMTP对象。

```
>>> smtpObj = smtplib.SMTP_SSL('smtp.gmail.com', 465)
```

> {注意}
>
> 
>
> 如果没有连接到因特网，Python将抛出socket.gaierror: [Errno 11004] getaddrinfo failed或类似的异常。

对于你的程序，TLS和SSL之间的区别并不重要。只需要知道你的SMTP服务器使用哪种加密标准，这样就知道如何连接它。在接下来的所有交互式环境示例中，smtpObj变量将包含smtplib.SMTP()或smtplib.SMTP_SSL()函数返回的SMTP对象。

### 16.2.2 发送SMTP的“Hello”消息

得到SMTP对象后，调用它的名字古怪的EHLO()方法，向SMTP电子邮件服务器“打招呼”。这种问候是SMTP中的第一步，对于建立到服务器的连接是很重要的。你不需要知道这些协议的细节。只要确保得到SMTP对象后，第一件事就是调用ehlo()方法，否则以后的方法调用会导致错误。下面是一个ehlo()调用和返回值的例子：

```
>>> smtpObj.ehlo()(250, b'mx.google.com at your service, [216.172.148.131]\nSIZE 35882577\n8BITMIME\nSTARTTLS\nENHANCEDSTATUSCODES\nCHUNKING')
```

如果在返回的元组中，第一项是整数250（SMTP中“成功”的代码），则问候成功了。

### 16.2.3 开始TLS加密

如果要连接到SMTP服务器的587端口（即使用TLS加密），接下来需要调用starttls()方法。这是为连接实现加密必须的步骤。如果要连接到465端口（使用SSL），加密已经设置好了，你应该跳过这一步。

下面是starttls()方法调用的例子：

```
>>> smtpObj.starttls()(220, b'2.0.0 Ready to start TLS')
```

starttls()让SMTP连接处于TLS模式。返回值220告诉你，该服务器已准备就绪。

### 16.2.4 登录到SMTP服务器

到SMTP服务器的加密连接建立后，可以调用login()方法，用你的用户名（通常是你的电子邮件地址）和电子邮件密码登录。

```
>>> smtpObj.login('my_email_address@gmail.com', 'MY_SECRET_PASSWORD')(235, b'2.7.0 Accepted')
```

传入电子邮件地址字符串作为第一个参数，密码字符串作为第二个参数。返回值235表示认证成功。如果密码不正确，Python会抛出smtplib. SMTPAuthenticationError异常。

将密码放在源代码中要当心。如果有人复制了你的程序，他们就能访问你的电子邮件账户！调用input()，让用户输入密码是一个好主意。每次运行程序时输入密码可能不方便，但这种方法不会在未加密的文件中留下你的密码，黑客或笔记本电脑窃贼不会轻易地得到它。

### 16.2.5 发送电子邮件

登录到电子邮件提供商的SMTP服务器后，可以调用的sendmail()方法来发送电子邮件。sendmail()方法调用看起来像这样：

```
>>> smtpObj.sendmail('my_email_address@gmail.com', 'recipient@example.com','Subject: So long.\nDear Alice, so long and thanks for all the fish. Sincerely,Bob'){}
```

sendmail()方法需要三个参数。

- 你的电子邮件地址字符串（电子邮件的“from”地址）。
- 收件人的电子邮件地址字符串，或多个收件人的字符串列表（作为“to”地址）。
- 电子邮件正文字符串。

电子邮件正文字符串必须以’Subject: \n’开头，作为电子邮件的主题行。’\n’换行符将主题行与电子邮件的正文分开。

sendmail()的返回值是一个字典。对于电子邮件传送失败的每个收件人，该字典中会有一个键值对。空的字典意味着对所有收件人已成功发送电子邮件。

> {Gmail应用程序专用密码!!}
>
> 
>
> Gmail有针对谷歌账户的附加安全功能，称为应用程序专用密码。如果当你的程序试图登录时，收到“需要应用程序专用密码”的错误信息，就必须在Python脚本设置这样一个密码。具体如何设置谷歌账户的应用程序专用密码，参见
>
> http://nostarch.com/automatestuff/
>
> 。

### 16.2.6 从SMTP服务器断开

确保在完成发送电子邮件时，调用quit()方法。这让程序从SMTP服务器断开。

```
>>> smtpObj.quit()(221, b'2.0.0 closing connection ko10sm23097611pbd.52 - gsmtp')
```

返回值221表示会话结束。

要复习连接和登录服务器、发送电子邮件和断开的所有步骤，请参阅 16.2节“发送电子邮件”。

## 16.3 IMAP

正如SMTP是用于发送电子邮件的协议，因特网消息访问协议（IMAP）规定了如何与电子邮件服务提供商的服务器通信，取回发送到你的电子邮件地址的电子邮件。Python带有一个imaplib模块，但实际上第三方的imapclient模块更易用。本章介绍了如何使用IMAPClient，完整的文档在http://imapclient.readthedocs.org/。

imapclient模块从IMAP服务器下载电子邮件，格式相当复杂。你很可能希望将它们从这种格式转换成简单的字符串。pyzmail模块替你完成解析这些邮件的辛苦工作。在http://www.magiksys.net/pyzmail/可以找到PyzMail的完整文档。

从终端窗口安装imapclient和pyzmail。附录A包含了如何安装第三方模块的步骤。

## 16.4 用IMAP获取和删除电子邮件

在Python中，查找和获取电子邮件是一个多步骤的过程，需要第三方模块imapclient和pyzmail。作为概述，这里有一个完整的例子，包括登录到IMAP服务器，搜索电子邮件，获取它们，然后从中提取电子邮件的文本。

```
>>> import imapclient>>> imapObj = imapclient.IMAPClient('imap.gmail.com', ssl=True)>>> imapObj.login('my_email_address@gmail.com', 'MY_SECRET_PASSWORD')'my_email_address@gmail.com Jane Doe authenticated (Success)'>>> imapObj.select_folder('INBOX', readonly=True)>>> UIDs = imapObj.search(['SINCE 05-Jul-2014'])>>> UIDs[40032, 40033, 40034, 40035, 40036, 40037, 40038, 40039, 40040, 40041]>>> rawMessages = imapObj.fetch([40041], ['BODY[]', 'FLAGS'])>>> import pyzmail>>> message = pyzmail.PyzMessage.factory(rawMessages[40041]['BODY[]'])>>> message.get_subject()'Hello!'>>> message.get_addresses('from')[('Edward Snowden', 'esnowden@nsa.gov')]>>> message.get_addresses('to')[(Jane Doe', 'jdoe@example.com')]>>> message.get_addresses('cc')[]>>> message.get_addresses('bcc')[]>>> message.text_part != NoneTrue>>> message.text_part.get_payload().decode(message.text_part.charset)'Follow the money.\r\n\r\n-Ed\r\n'>>> message.html_part != NoneTrue>>> message.html_part.get_payload().decode(message.html_part.charset)'< div dir="ltr">< div>So long, and thanks for all the fish!< br>< br>< /div>-Al< br>< /div>\r\n'>>> imapObj.logout()
```

你不必记住这些步骤。在详细介绍每一步之后，你可以回来看这个概述，加强记忆。

### 16.4.1 连接到IMAP服务器

就像你需要一个SMTP对象连接到SMTP服务器并发送电子邮件一样，你需要一个IMAPClient对象，连接到IMAP服务器并接收电子邮件。首先，你需要电子邮件服务提供商的IMAP服务器域名。这和SMTP服务器的域名不同。表16-2列出了几个流行的电子邮件服务提供商的IMAP服务器。

表16-2 电子邮件提供商及其IMAP服务器

| 提供商                  | IMAP服务器域名          |
| ----------------------- | ----------------------- |
| Gmail                   | *imap.gmail.com*        |
| Outlook.com/Hotmail.com | *imap-mail.outlook.com* |
| Yahoo Mail              | *imap.mail.yahoo.com*   |
| AT&T                    | *imap.mail.att.net*     |
| Comcast                 | *imap.comcast.net*      |
| Verizon                 | *incoming.verizon.net*  |

得到IMAP服务器域名后，调用imapclient.IMAPClient()函数，创建一个IMAPClient对象。大多数电子邮件提供商要求SSL加密，传入SSL= TRUE关键字参数。在交互式环境中输入以下代码（使用你的提供商的域名）：

```
>>> import imapclient>>> imapObj = imapclient.IMAPClient('imap.gmail.com', ssl=True)
```

在接下来的小节里所有交互式环境的例子中，imapObj变量将包含imapclient.IMAPClient()函数返回的IMAPClient对象。在这里，客户端是连接到服务器的对象。

### 16.4.2 登录到IMAP服务器

取得IMAPClient对象后，调用它的login()方法，传入用户名（这通常是你的电子邮件地址）和密码字符串。

```
>>> imapObj.login('my_email_address@gmail.com', 'MY_SECRET_PASSWORD')'my_email_address@gmail.com Jane Doe authenticated (Success)'
```

要记住，永远不要直接在代码中写入密码！应该让程序从input()接受输入的密码。

如果IMAP服务器拒绝用户名/密码的组合，Python会抛出imaplib.error异常。对于Gmail账户，你可能需要使用应用程序专用的密码。详细信息请参阅16.2.5节中的“Gmail应用程序专用密码”。

### 16.4.3 搜索电子邮件

登录后，实际获取你感兴趣的电子邮件分为两步。首先，必须选择要搜索的文件夹。然后，必须调用IMAPClient对象的search()方法，传入IMAP搜索关键词字符串。

### 16.4.4 选择文件夹

几乎每个账户默认都有一个INBOX文件夹，但也可以调用IMAPClient对象的list_folders()方法，获取文件夹列表。这将返回一个元组的列表。每个元组包含一个文件夹的信息。输入以下代码，继续交互式环境的例子：

```
>>> import pprint>>> pprint.pprint(imapObj.list_folders())[(('\\\HasNoChildren',), '/', 'Drafts'), (('\\\HasNoChildren',), '/', 'Filler'), (('\\\HasNoChildren',), '/', 'INBOX'), (('\\\HasNoChildren',), '/', 'Sent'),--snip-- (('\\\HasNoChildren', '\\\Flagged'), '/', '[Gmail]/Starred'), (('\\\HasNoChildren', '\\\Trash'), '/', '[Gmail]/Trash')]
```

如果你有一个Gmail账户，这就是输出可能的样子（Gmail将文件夹称为label，但它们的工作方式与文件夹相同）。每个元组的三个值，例如 ((‘\\HasNoChildren’,), ‘/‘, ‘INBOX’)，解释如下：

- 该文件夹的标志的元组（这些标志代表到底是什么超出了本书的讨论范围，你可以放心地忽略该字段）。
- 名称字符串中用于分隔父文件夹和子文件夹的分隔符。
- 该文件夹的全名。

要选择一个文件夹进行搜索，就调用IMAPClient对象的select_folder()方法，传入该文件夹的名称字符串。

```
>>> imapObj.select_folder('INBOX', readonly=True)
```

可以忽略select_folder()的返回值。如果所选文件夹不存在，Python会抛出imaplib.error异常。

readonly=True关键字参数可以防止你在随后的方法调用中，不小心更改或删除该文件夹中的任何电子邮件。除非你想删除的电子邮件，否则将readonly设置为True总是个好主意。

### 16.4.5 执行搜索

文件夹选中后，就可以用IMAPClient对象的search()方法搜索电子邮件。search()的参数是一个字符串列表，每一个格式化为IMAP搜索键。表16-3介绍了各种搜索键。

表16-3 IMAP搜索键

| 搜索键                                                       | 含义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ‘ALL’                                                        | 返回该文件夹中的所有邮件。如果你请求一个大文件夹中的所有消息，可能会遇到imaplib的大小限制。参见16.4.6小节“大小限制” |
| ‘BEFORE *date*‘, ‘ON *date*‘, ‘SINCE *date*‘                 | 这三个搜索键分别返回给定date之前、当天和之后IMAP服务器接收的消息。日期的格式必须像05-Jul-2015。此外，虽然’SINCE 05-Jul-2015’将匹配7月5日当天和之后的消息，但’BEFORE 05-Jul-2015’仅匹配7月5日之前的消息，不包括7月5日当天 |
| ‘SUBJECT *string*‘, ‘BODY *string*‘, ‘TEXT *string*‘         | 分别返回string出现在主题、正文、主题或正文中的消息。如果string中有空格，就使用双引号：’TEXT “search with spaces”‘ |
| ‘FROM *string*‘, ‘TO *string*‘, ‘CC *string*‘, ‘BCC *string*‘ | 返回所有消息，其中string分别出现在“from”邮件地址，“to”邮件地址，“cc”（抄送）地址，或“bcc”（密件抄送）地址中。 如果string中有多个电子邮件地址，就用空格将它们分开，并使用双引号： ‘CC “*[firstcc@example.com](mailto:firstcc@example.com) [secondcc@example.com](mailto:secondcc@example.com)*“‘ |
| ‘SEEN’, ‘UNSEEN’                                             | 分别返回包含和不包含\ Seen标记的所有信息。如果电子邮件已经被fetch()方法调用访问（稍后描述），或者你曾在电子邮件程序或网络浏览器中点击过它，就会有\ Seen标记。比较常用的说法是电子邮件“已读”，而不是“已看”，但它们的意思一样。 |
| ‘ANSWERED’, ‘UNANSWERED’                                     | 分别返回包含和不包含\ Answered标记的所有消息。如果消息已答复，就会有\ Answered标记 |
| ‘DELETED’, ‘UNDELETED’                                       | 分别返回包含和不包含*\Deleted*标记的所有信息。用delete_messages()方法删除的邮件就会有*\Deleted*标记，直到调用expunge()方法才会永久删除（请参阅16.4.10节“删除电子邮件”）。请注意，一些电子邮件提供商，例如Gmail，会自动清除邮件 |
| ‘DRAFT’, ‘UNDRAFT’                                           | 分别返回包含和不包含\ Draft标记的所有消息。草稿邮件通常保存在单独的草稿文件夹中，而不是在收件箱中 |
| ‘FLAGGED’, ‘UNFLAGGED’                                       | 分别返回包含和不包含*\Flagged*标记的所有消息。这个标记通常用来标记电子邮件为“重要”或“紧急” |
| ‘LARGER *N*‘, ‘SMALLER *N*‘                                  | 分别返回大于或小于N个字节的所有消息                          |
| ‘NOT *search-key*‘                                           | 返回搜索键不会返回的那些消息                                 |
| ‘OR *search-key1 search-key2*‘                               | 返回符合第一个或第二个搜索键的消息                           |

请注意，在处理标志和搜索键方面，某些IMAP服务器的实现可能稍有不同。可能需要在交互式环境中试验一下，看看它们实际的行为如何。

在传入search()方法的列表参数中，可以有多个IMAP搜索键字符串。返回的消息将匹配所有的搜索键。如果想匹配任何一个搜索键，使用OR搜索键。对于NOT和OR搜索键，它们后边分别跟着一个和两个完整的搜索键。

下面是search()方法调用的一些例子，以及它们的含义：

**imapObj.search**(**[‘ALL’]**) 返回当前选定的文件夹中的每一个消息。

**imapObj.search**(**[‘ON 05-Jul-2015’]**)返回在2015年7月5日发送的每个消息。

**imapObj.search([‘SINCE 01-Jan-2015’, ‘BEFORE 01-Feb-2015’, ‘UNSEEN’])**返回2015年1月发送的所有未读消息（注意，这意味着从1月1日直到2月1日，但不包括2月1日）。

**imapObj.search([‘SINCE 01-Jan-2015’, ‘FROM [alice@example.com](mailto:alice@example.com)’])**返回自2015年开始以来，发自[alice@example.com](mailto:alice@example.com)的消息。

**imapObj.search([‘SINCE 01-Jan-2015’, ‘NOT FROM [alice@example.com](mailto:alice@example.com)’])**返回自2015年开始以来，除[alice@example.com](mailto:alice@example.com)外，其他所有人发来的消息。

**imapObj.search([‘OR FROM [alice@example.com](mailto:alice@example.com) FROM [bob@example.com](mailto:bob@example.com)’])**返回发自[alice@example.com](mailto:alice@example.com)或[bob@example.com](mailto:bob@example.com)的所有信息。

**imapObj.search([‘FROM [alice@example.com](mailto:alice@example.com)’, ‘FROM [bob@example.com](mailto:bob@example.com)’])恶作剧**例子！该搜索不会返回任何消息，因为消息必须匹配所有搜索关键词。因为只能有一个“from”地址，所以一条消息不可能既来自[alice@example.com](mailto:alice@example.com)，又来自[bob@example.com](mailto:bob@example.com)。

search()方法不返回电子邮件本身，而是返回邮件的唯一整数ID（UID）。然后，可以将这些UID传入fetch()方法，获得邮件内容。

输入以下代码，继续交互式环境的例子：

```
>>> UIDs = imapObj.search(['SINCE 05-Jul-2015'])>>> UIDs[40032, 40033, 40034, 40035, 40036, 40037, 40038, 40039, 40040, 40041]
```

这里，search()返回的消息ID列表（针对7月5日以来接收的消息）保存在UIDs中。计算机上返回的UIDs列表与这里显示的不同，它们对于特定的电子邮件账户是唯一的。如果你稍后将UID传递给其他函数调用，请用你收到的UID值，而不是本书例子中打印的。

### 16.4.6 大小限制

如果你的搜索匹配大量的电子邮件，Python可能抛出异常imaplib.error: got more than 10000 bytes。如果发生这种情况，必须断开并重连IMAP服务器，然后再试。

这个限制是防止Python程序消耗太多内存。遗憾的是，默认大小限制往往太小。可以执行下面的代码，将限制从10000字节改为10000000字节：

```
>>> import imaplib>>> imaplib._MAXLINE = 10000000
```

这应该能避免该错误消息再次出现。也许要在你写的每一个IMAP程序中加上这两行。

### 16.4.7 取邮件并标记为已读

得到UID的列表后，可以调用IMAPClient对象的fetch()方法，获得实际的电子邮件内容。

UID列表是fetch()的第一个参数。第二个参数应该是[‘BODY[]’]，它告诉fetch()下载UID列表中指定电子邮件的所有正文内容。

> {使用IMAPClient的gmail_search()方法!!}
>
> 
>
> 如果登录到imap.gmail.com服务器来访问Gmail账户，IMAPClient对象提供了一个额外的搜索函数，模拟Gmail网页顶部的搜索栏，如图16-1中高亮的部分所示。
>
> 
>
> ![..\16-0403yds排版\16-0403yds排版\Links\Links\automate16-gmailsearch2.png{397}](http://write.epubit.com:9000/api/storage/getbykey/original?key=16080396c52e04a086ce)
>
> 图16-1 在Gmail网页顶部的搜索栏
>
> 除了用IMAP搜索键搜索，可以使用Gmail更先进的搜索引擎。Gmail在匹配密切相关的单词方面做得很好（例如，搜索driving也会匹配drive和drove），并按照匹配的程度对搜索结果排序。也可以使用Gmail的高级搜索操作符（更多信息请参见http://nostarch.com/automatestuff/）。如果登录到Gmail账户，向gmail_search()方法传入搜索条件，而不是search()方法，就像下面交互式环境的例子：
>
> ```
> >>> UIDs = imapObj.gmail_search('meaning of life')>> UIDs[42]
> ```
>
> 啊，是的，那封电子邮件包含了生命的意义！我一直在期待。

让我们继续交互式环境的例子。

```
>>> rawMessages = imapObj.fetch(UIDs, ['BODY[]'])>>> import pprint>>> pprint.pprint(rawMessages){40040: {'BODY[]': 'Delivered-To: my_email_address@gmail.com\r\n'                   'Received: by 10.76.71.167 with SMTP id '--snip--                   '\r\n'                   '------=_Part_6000970_707736290.1404819487066--\r\n',         'SEQ': 5430}}
```

导入 pprint，将 fetch()的返回值（保存在变量 rawMessages 中）传入pprint.pprint()，“漂亮打印”它。你会看到，这个返回值是消息的嵌套字典，其中以UID作为键。每条消息都保存为一个字典，包含两个键：’BODY[]’和’SEQ’。’BODY[]’键映射到电子邮件的实际正文。’SEQ’键是序列号，它与UID的作用类似。你可以放心地忽略它。

正如你所看到的，在’BODY[]’键中的消息内容是相当难理解的。这种格式称为RFC822，是专为IMAP服务器读取而设计的。但你并不需要理解RFC 822格式，本章稍后的pyzmail模块将替你来理解它。

如果你选择一个文件夹进行搜索，就用readonly=True关键字参数来调用select_ folder()。这样做可以防止意外删除电子邮件，但这也意味着你用fetch()方法获取邮件时，它们不会标记为已读。如果确实希望在获取邮件时将它们标记已读，就需要将readonly=False传入select_folder()。如果所选文件夹已处于只读模式，可以用另一个 select_folder()调用重新选择当前文件夹，这次用readonly=False关键字参数：

```
>>> imapObj.select_folder('INBOX', readonly=False)
```

### 16.4.8 从原始消息中获取电子邮件地址

对于只想读邮件的人来说，fetch()方法返回的原始消息仍然不太有用。pyzmail模块解析这些原始消息，将它们作为PyzMessage对象返回，使邮件的主题、正文、“收件人”字段、“发件人”字段和其他部分能用Python代码轻松访问。

用下面的代码继续交互式环境的例子（使用你自己的邮件账户的UID，而不是这里显示的）：

```
>>> import pyzmail>>> message = pyzmail.PyzMessage.factory(rawMessages[40041]['BODY[]'])
```

首先，导入pyzmail。然后，为了创建一个电子邮件的PyzMessage对象，调用pyzmail.PeekMessage.factory()函数，并传入原始邮件的’BODY[]’部分。结果保存在message中。现在，message中包含一个PyzMessage对象，它有几个方法，可以很容易地获得的电子邮件主题行，以及所有发件人和收件人的地址。get_subject()方法将主题返回为一个简单字符串。get_addresses()方法针对传入的字段，返回一个地址列表。例如，该方法调用可能像这样：

```
>>> message.get_subject()'Hello!'>>> message.get_addresses('from')[('Edward Snowden', 'esnowden@nsa.gov')]>>> message.get_addresses('to')[(Jane Doe', 'my_email_address@gmail.com')]>>> message.get_addresses('cc')[]>>> message.get_addresses('bcc')[]
```

请注意，get_addresses()的参数是’from’、’to’、’cc’或 ‘bcc’。get_addresses()的返回值是一个元组列表。每个元组包含两个字符串：第一个是与该电子邮件地址关联的名称，第二个是电子邮件地址本身。如果请求的字段中没有地址，get_addresses()返回一个空列表。在这里，’cc’抄送和’bcc’密件抄送字段都没有包含地址，所以返回空列表。

### 16.4.9 从原始消息中获取正文

电子邮件可以是纯文本、HTML 或两者的混合。纯文本电子邮件只包含文本，而HTML电子邮件可以有颜色、字体、图像和其他功能，使得电子邮件看起来像一个小网页。如果电子邮件仅仅是纯文本，它的PyzMessage对象会将html_part属性设为None。同样，如果电子邮件只是HTML，它的PyzMessage对象会将text_part属性设为None。

否则，text_part或html_part将有一个get_payload()方法，将电子邮件的正文返回为bytes数据类型（bytes数据类型超出了本书的范围）。但是，这仍然不是我们可以使用的字符串。啊！最后一步对get_payload()返回的bytes值调用decode()方法。decode()方法接受一个参数：这条消息的字符编码，保存在text_part.charset或html_part.charset属性中。最后，这返回了邮件正文的字符串。

输入以下代码，继续交互式环境的例子：

```
❶ >>> message.text_part != None　True　>>> message.text_part.get_payload().decode(message.text_part.charset)❷ 'So long, and thanks for all the fish!\r\n\r\n-Al\r\n'❸ >>> message.html_part != None　True❹ >>> message.html_part.get_payload().decode(message.html_part.charset)　'< div dir="ltr">< div>So long, and thanks for all the fish!< br>< br>< /div>-Al　< br>< /div>\r\n'
```

我们正在处理的电子邮件包含纯文本和HTML内容，因此保存在message中的PyzMessage对象的text_part和html_part属性不等于None❶❸。对消息的text_part调用get_payload()，然后在bytes值上调用decode()，返回电子邮件的文本版本的字符串❷。对消息的html_part调用get_payload()和decode()，返回电子邮件的HTML版本的字符串❹。

### 16.4.10 删除电子邮件

要删除电子邮件，就向IMAPClient对象的delete_messages()方法传入一个消息UID的列表。这为电子邮件加上\Deleted标志。调用expunge()方法，将永久删除当前选中的文件夹中带\Deleted标志的所有电子邮件。请看下面的交互式环境的例子：

```
❶ >>> imapObj.select_folder('INBOX', readonly=False)❷ >>> UIDs = imapObj.search(['ON 09-Jul-2015'])　>>> UIDs　[40066]　>>> imapObj.delete_messages(UIDs)❸ {40066: ('\\\Seen', '\\\Deleted')}　>>> imapObj.expunge()　('Success', [(5452, 'EXISTS')])
```

这里，我们调用了IMAPClient对象的select_folder()方法，传入’INBOX’作为第一个参数，选择了收件箱。我们也传入了关键字参数readonly=False，这样我们就可以删除电子邮件❶。我们搜索收件箱中的特定日期收到的消息，将返回的消息ID保存在UIDs中❷。调用delete_message()并传入UIDs，返回一个字典，其中每个键值对是一个消息 ID 和消息标志的元组，它现在应该包含\Deleted标志❸。然后调用expunge()，永久删除带\Deleted标志的邮件。如果清除邮件没有问题，就返回一条成功信息。请注意，一些电子邮件提供商，如Gmail，会自动清除用delete_messages()删除的电子邮件，而不是等待来自IMAP客户端的expunge命令。

### 16.4.11 从IMAP服务器断开

如果程序已经完成了获取和删除电子邮件，就调用IMAPClient的logout()方法，从IMAP服务器断开连接。

```
>>> imapObj.logout()
```

如果程序运行了几分钟或更长时间，IMAP服务器可能会超时，或自动断开。在这种情况下，接下来程序对IMAPClient对象的方法调用会抛出异常，像下面这样：

```
imaplib.abort: socket error: [WinError 10054] An existing connection wasforcibly closed by the remote host
```

在这种情况下，程序必须调用imapclient.IMAPClient()，再次连接。

哟！齐活了。要跳过很多圈圈，但你现在有办法让Python程序登录到一个电子邮件账户，并获取电子邮件。需要回忆所有步骤时，你可以随时参考16.4节“用IMAP获取和删除电子邮件”。

## 16.5 项目：向会员发送会费提醒电子邮件

假定你一直“自愿”为“强制自愿俱乐部”记录会员会费。这确实是一项枯燥的工作，包括维护一个电子表格，记录每个月谁交了会费，并用电子邮件提醒那些没交的会员。不必你自己查看电子表格，而是向会费超期的会员复制和粘贴相同的电子邮件。你猜对了，让我们编写一个脚本，帮你完成任务。

在较高的层面上，下面是程序要做的事：

- 从Excel电子表格中读取数据。
- 找出上个月没有交费的所有会员。
- 找到他们的电子邮件地址，向他们发送针对个人的提醒。

这意味着代码需要做到以下几点：

- 用openpyxl模块打开并读取Excel文档的单元格（处理Excel文件参见第12章）。
- 创建一个字典，包含会费超期的会员。
- 调用smtplib.SMTP()、ehlo()、starttls()和login()，登录SMTP服务器。
- 针对会费超期的所有会员，调用sendmail()方法，发送针对个人的电子邮件提醒。

打开一个新的文件编辑器窗口，并保存为sendDuesReminders.py。

### 第1步：打开Excel文件

假定用来记录会费支付的 Excel 电子表格看起来如图 16-2 所示，放在名为duesRecords.xlsx的文件中。可以从http://nostarch.com/automatestuff/下载该文件。

![..\16-0403yds排版\16-0403yds排版\Links\Links\automate16-memberdues.png{342}](http://write.epubit.com:9000/api/storage/getbykey/original?key=16087ab07f8a0eaf0b12)

图16-2 记录会员会费支付电子表格

该电子表格中包含每个成员的姓名和电子邮件地址。每个月有一列，记录会员的付款状态。在成员交纳会费后，对应的单元格就记为paid。

该程序必须打开duesRecords.xlsx，通过调用get_highest_column()方法，弄清楚最近一个月的列（可以参考第12章，了解用openpyxl模块访问Excel电子表格文件单元格的更多信息）。在文件编辑器窗口中输入以下代码：

```
　#! python3　# sendDuesReminders.py - Sends emails based on payment status in spreadsheet.　import openpyxl, smtplib, sys　# Open the spreadsheet and get the latest dues status.❶ wb = openpyxl.load_workbook('duesRecords.xlsx')❷ sheet = wb.get_sheet_by_name('Sheet1')❸ lastCol = sheet.get_highest_column()❹ latestMonth = sheet.cell(row=1, column=lastCol).value　# TODO: Check each member's payment status.　# TODO: Log in to email account.　# TODO: Send out reminder emails.
```

导入openpyxl、smtplib和sys模块后，我们打开duesRecords.xlsx文件，将得到的Workbook对象保存在wb中❶。然后，取得Sheet 1，将得到的Worksheet对象保存在sheet中❷。既然有了Worksheet对象，就可以访问行、列和单元格。我们将最后一列保存在lastCol中❸，然后用行号1和lastCol来访问应该记录着最近月份的单元格。取得该单元格的值，并保存在latestMonth 中❹。

### 第2步：查找所有未付成员

一旦确定了最近一个月的列数（保存在lastCol中），就可以循环遍历第一行（这是列标题）之后的所有行，看看哪些成员在该月会费的单元格中写着paid。如果会员没有支付，就可以从列1和2中分别抓取成员的姓名和电子邮件地址。这些信息将放入unpaidMembers字典，它记录最近一个月没有交费的所有成员。将以下代码添加到sendDuesReminder.py中。

```
　#! python3　# sendDuesReminders.py - Sends emails based on payment status in spreadsheet.　--snip--　# Check each member's payment status.　unpaidMembers = {}❶ for r in range(2, sheet.get_highest_row() + 1):❷      payment = sheet.cell(row=r, column=lastCol).value　    if payment != 'paid':❸           name = sheet.cell(row=r, column=1).value❹           email = sheet.cell(row=r, column=2).value❺           unpaidMembers[name] = email
```

这段代码设置了一个空字典unpaidMembers，然后循环遍历第一行之后所有的行❶。对于每一行，最近月份的值保存在payment中❷。如果payment不等于’paid’，则第一列的值保存在name中❸，第二列的值保存在email中❹，name和email添加到unpaidMembers中❺。

### 第3步：发送定制的电子邮件提醒

得到所有未付费成员的名单后，就可以向他们发送电子邮件提醒了。将下面的代码添加到程序中，但要代入你的真实电子邮件地址和提供商的信息：

```
#! python3# sendDuesReminders.py - Sends emails based on payment status in spreadsheet.--snip--# Log in to email account.smtpObj = smtplib.SMTP('smtp.gmail.com', 587)smtpObj.ehlo()smtpObj.starttls()smtpObj.login('my_email_address@gmail.com', sys.argv[1])
```

调用smtplib.SMTP()并传入提供商的域名和端口，创建一个SMTP对象。调用ehlo()和starttls()，然后调用login()，并传入你的电子邮件地址和sys.argv[1]，其中保存着你的密码字符串。在每次运行程序时，将密码作为命令行参数输入，避免在源代码中保存密码。

程序登录到你的电子邮件账户后，就应该遍历unpaidMembers字典，向每个会员的电子邮件地址发送针对个人的电子邮件。将以下代码添加到sendDuesReminders.py：

```
　#! python3　# sendDuesReminders.py - Sends emails based on payment status in spreadsheet.　--snip--　# Send out reminder emails.　for name, email in unpaidMembers.items():❶      body = "Subject: %s dues unpaid.\nDear %s,\nRecords show that you have not　paid dues for %s. Please make this payment as soon as possible. Thank you!'" %　(latestMonth, name, latestMonth)❷      print('Sending email to %s...' % email)❸      sendmailStatus = smtpObj.sendmail('my_email_address@gmail.com', email, body)❹      if sendmailStatus != {}:　        print('There was a problem sending email to %s: %s' % (email,　        sendmailStatus))　smtpObj.quit()
```

这段代码循环遍历unpaidMembers中的姓名和电子邮件。对于每个没有付费的成员，我们用最新的月份和成员的名称，定制了一条消息，并保存在body中❶。我们打印输出，表示正在向这个会员的电子邮件地址发送电子邮件❷。然后调用sendmail()，向它传入地址和定制的消息❸。返回值保存在sendmailStatus中。

回忆一下，如果SMTP服务器在发送某个电子邮件时报告错误，sendmail()方法将返回一个非空的字典值。for循环最后部分在❹行检查返回的字典是否非空，如果非空，则打印收件人的电子邮件地址以及返回的字典。

程序完成发送所有电子邮件后，调用quit()方法，与SMTP服务器断开连接。

如果运行该程序，输出会像这样：

```
Sending email to alice@example.com...Sending email to bob@example.com...Sending email to eve@example.com...
```

收件人将收到如图16-3所示的电子邮件。

![..\16-0403yds排版\16-0403yds排版\Links\Links\16-3.tif{421}](http://write.epubit.com:9000/api/storage/getbykey/original?key=160836703f3bb046bf8a)

图16-3 从sendDuesReminders.py自动发送的电子邮件

## 16.6 用Twilio发送短信

大多数人更可能靠近自己的手机，而不是自己的电脑，所以与电子邮件相比，短信发送通知可能更直接、可靠。此外，短信的长度较短，让人更有可能阅读它们。

在本节中，你将学习如何注册免费的Twilio服务，并用它的Python模块发送短信。Twilio是一个SMS网关服务，这意味着它是一种服务，让你通过程序发送短信。虽然每月发送多少短信会有限制，并且文本前面会加上Sent from a Twilio trial account，但这项试用服务也许能满足你的个人程序。免费试用没有限期，不必以后升级到付费的套餐。

Twilio不是唯一的SMS网关服务。如果你不喜欢使用Twilio，可以在线搜索free sms gateway、python sms api，甚至twilio alternatives，寻找替代服务。

注册Twilio账户之前，先安装twilio模块。附录A详细介绍了如何安装第三方模块。

本节特别针对美国。Twilio 确实也在美国以外的国家提供手机短信服务，本书并不包括这些细节。但twilio 模块及其功能，在美国以外的国家也能用。更多信息请参见http://twilio.com/。

### 16.6.1 注册Twilio账号

访问http://twilio.com/并填写注册表单。注册了新账户后，你需要验证一个手机号码，短信将发给该号码（这项验证是必要的，防止有人利用该服务向任意的手机号码发送垃圾短信）。

收到验证号码短信后，在Twilio网站上输入它，证明你拥有要验证的手机。现在，就可以用twilio模块向这个电话号码发送短信了。

Twilio提供的试用账户包括一个电话号码，它将作为短信的发送者。你将需要两个信息：你的账户SID和AUTH（认证）标志。在登录Twilio账户时，可以在Dashboard页面上找到这些信息。从Python程序登录时，这些值将作为你的Twilio用户名和密码。

### 16.6.2 发送短信

一旦安装了twilio模块，注册了Twilio账号，验证了你的手机号码，登记了Twilio电话号码，获得了账户的SID和auth标志，你就终于准备好通过Python脚本向你自己发短信了。

与所有的注册步骤相比，实际的Python代码很简单。保持计算机连接到因特网，在交互式环境中输入以下代码，用你的真实信息替换accountSID、authToken、myTwilioNumber和myCellPhone变量的值：

```
❶ >>> from twilio.rest import TwilioRestClient　>>> accountSID = 'ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'　>>> authToken = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'❷ >>> twilioCli = TwilioRestClient(accountSID, authToken)　>>> myTwilioNumber = '+14955551234'　>>> myCellPhone = '+14955558888'❸ >>> message = twilioCli.messages.create(body='Mr. Watson - Come here - I want　to see you.', from_=myTwilioNumber, to=myCellPhone)
```

键入最后一行后不久，你会收到一条短信，内容为：Sent from your Twilio trial account - Mr. Watson - Come here – I want to see you.。

因为twilio模块的设计方式，导入它时需要使用from twilio.rest import TwilioRestClient，而不仅仅是import twilio❶。将账户的SID保存在accountSID，认证标志保存在authToken中，然后调用TwilioRestClient()，并传入accountSID和authToken。TwilioRestClient()调用返回一个TwilioRestClient对象❷。该对象有一个message属性，该属性又有一个create()方法，可以用来发送短信。正是这个方法，将告诉Twilio的服务器发送短信。将你的Twilio号码和手机号码分别保存在myTwilioNumber和myCellPhone中，然后调用create()，传入关键字参数，指明短信的正文、发件人的号码（myTwilioNumber），以及收信人的电话号码（myCellPhone）❸

create()方法返回的Message对象将包含已发送短信的相关信息。输入以下代码，继续交互式环境的例子：

```
>>> message.to'+14955558888'>>> message.from_'+14955551234'>>> message.body'Mr. Watson - Come here - I want to see you.'
```

to、from*和body属性应该分别保存了你的手机号码、Twilio号码和消息。请注意，发送手机号码是在from*属性中，末尾有一个下划线，而不是from。这是因为from是一个Python关键字（例如，你在from modulename import *形式的import语句中见过它），所以它不能作为一个属性名。输入以下代码，继续交互式环境的例子：

```
>>> message.status'queued'>>> message.date_createddatetime.datetime(2015, 7, 8, 1, 36, 18)>>> message.date_sent == NoneTrue
```

status 属性应该包含一个字符串。如果消息被创建和发送，date_created 和date_sent属性应该包含一个datetime对象。如果已收到短信，而status属性却设置为’queued’，date_sent属性设置为None，这似乎有点奇怪。这是因为你先将Message对象记录在message变量中，然后短信才实际发送。你需要重新获取Message对象，查看它最新的status和date_sent。每个Twilio消息都有唯一的字符串ID（SID），可用于获取Message对象的最新更新。输入以下代码，继续交互式环境的例子：

```
　>>> message.sid　'SM09520de7639ba3af137c6fcb7c5f4b51'❶ >>> updatedMessage = twilioCli.messages.get(message.sid)　>>> updatedMessage.status　'delivered'　>>> updatedMessage.date_sent　datetime.datetime(2015, 7, 8, 1, 36, 18)
```

输入message.sid将显示这个消息的SID。将这个SID传入Twilio客户端的get()方法❶，你可以取得一个新的Message对象，包含最新的信息。在这个新的Message对象中，status和date_sent属性是正确的。

status属性将设置为下列字符串之一：’queued’、’sending’、’sent’、’delivered’、’undelivered’或’failed’。这些状态不言自明，但对于更准确的细节，请查看[http://nostarch](http://nostarch/). com/automatestuff/的资源。

> {用Python接收短信!!}
>
> 
>
> 遗憾的是，用Twilio接收短信比发送短信更复杂一些。Twilio需要你有一个网站，运行自己的Web应用程序。这已超出了本书的范围，但你可以在本书的资源中找到更多细节（
>
> http://nostarch.com/automatestuff/）。

## 16.7 项目：“只给我发短信”模块

最常用你的程序发短信的人可能就是你。当你远离计算机时，短信是通知你自己的好方式。如果你已经用程序自动化了一个无聊的任务，它需要运行几小时，你可以在它完成时，让它用短信通知你。或者可以定期运行某个程序，它有时需要与你联系，例如天气检查程序，用短信提醒你带伞。

举一个简单的例子，下面是一个Python小程序，包含了textmyself()函数，它将传入的字符串参数作为短信发出。打开一个新的文件编辑器窗口，输入以下代码，用自己的信息替换帐户SID，认证标志和电话号码。将它保存为textMyself.py。

```
　#! python3　# textMyself.py - Defines the textmyself() function that texts a message　# passed to it as a string.　# Preset values:　accountSID = 'ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'　authToken = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'　myNumber = '+15559998888'　twilioNumber = '+15552225678'　from twilio.rest import TwilioRestClient❶ def textmyself(message):❷      twilioCli = TwilioRestClient(accountSID, authToken)❸      twilioCli.messages.create(body=message, from_=twilioNumber, to=myNumber)
```

该程序保存了账户的SID、认证标志、发送号码及接收号码。然后它定义了textmyself()，接收参数❶，创建TwilioRestClient对象❷，并用你传入的消息调用create()❸。

如果你想让其他程序使用textmyself()函数，只需将textMyself.py文件和Python的可执行文件放在同一个文件夹中（Windows上是C:\Python34，OS X上是/usr/local/lib/python3.4，Linux上是/usr/bin/python3）。现在，你可以在其他程序中使用该函数。只要想在程序中发短信给你，就添加以下代码：

```
import textmyselftextmyself.textmyself('The boring task is finished.')
```

注册Twilio和编写短信代码只要做一次。在此之后，从任何其他程序中发短信，只要两行代码。

## 16.8 小结

通过因特网和手机网络，我们用几十种不同的方式相互通信，但以电子邮件和短信为主。你的程序可以通过这些渠道沟通，这给它们带来强大的新通知功能。甚至可以编程运行在不同的计算机上，相互直接通过电子邮件能信，一个程序用SMTP发送电子邮件，另一个用IMAP收取。

Python 的 smtplib 提供了一些函数，利用 SMTP，通过电子邮件提供商的SMTP服务器发送电子邮件。同样，第三方的imapclient和pyzmail模块让你访问IMAP服务器，并取回发送给你的电子邮件。虽然IMAP比SMTP复杂一些，但它也相当强大，允许你搜索特定电子邮件、下载它们、解析它们，提取主题和正文作为字符串值。

短信与电子邮件有点不同，因为它不像电子邮件，发送短信不仅需要互联网连接。好在，像Twilio这样的服务提供了模块，允许你通过程序发送短信。一旦通过了初始设置过程，就能够只用几行代码来发送短信。掌握了这些模块，就可以针对特定的情况编程，在这些情况下发送通知或提醒。现在，你的程序将超越运行它们的计算机！

**本文摘自《Python编程快速上手 让繁琐工作自动化》**

## 参考文档

[用Python发送邮件，需要这样三步](https://www.epubit.com/articleDetails?id=11c1d7eaa14e4f0dbfac84e307ed166d)

