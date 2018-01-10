# Hyperledger Fabric v1.0 官方文档（中文翻译） #

## 项目简介 ##

本文档是 [Hyperledger Fabric v1.0.5](https://github.com/hyperledger/fabric/tree/014d6befcf67f3787bb3d67ff34e1a98dc6aec5f) 官方文档的简体中文翻译版本， 翻译工作正在进行中， 欢迎加入。

Email: boyu.liu@gmail.com

## 许可协议 ##

本文档采用[知识共享署名 4.0 国际许可协议](http://creativecommons.org/licenses/by/4.0/)进行许可。  
[![知识共享许可协议](https://i.creativecommons.org/l/by/4.0/88x31.png "知识共享许可协议")](http://creativecommons.org/licenses/by/4.0/)  

## 在线阅读 ##

Read the Docs: [Hyperledger Fabric v1.0 官方文档（中文翻译）](http://hyperledger-fabric-docs-zh-cn.readthedocs.io/zh_CN/latest/)  
![Read the Docs](https://readthedocs.org/projects/hyperledger-fabric-docs-zh-cn/badge/)  

Read the Docs: [Hyperledger Fabric v1.0 官方文档（英文原版）](http://hyperledger-fabric.readthedocs.io/en/latest/)  
![Read the Docs](https://readthedocs.org/projects/hyperledger-fabric/badge/)  

## 项目计划 ##

| 章节 | 文件 | 行数 | 状态 |
| --- | --- | --- | --- |
| GETTING STARTED                                |                          |      |   |
| Prerequisites                                  | prereqs                  |  161 | - |
| Getting Started                                | getting_started          |   77 | - |
| Hyperledger Fabric Samples                     | samples                  |   94 | - |
| KEY CONCEPTS                                   |                          |      |   |
| Introduction                                   | blockchain               |  249 | - |
| Hyperledger Fabric Capabilities                | capabilities             |   76 | - |
| Hyperledger Fabric Model                       | fabric_model             |  161 | DONE |
| Use Cases                                      | usecases                 |   11 | DONE |
| TUTORIALS                                      |                          |      |   |
| Building Your First Network                    | build_network            | 1058 | DOING |
| Writing Your First Application                 | write_first_app          |  506 | - |
| Chaincode Tutorials                            | chaincode                |   33 | - |
| Chaincode for Developers                       | chaincode4ade            |  522 | - |
| Chaincode for Operators                        | chaincode4noah           |  416 | - |
| Videos                                         | videos                   |   17 | - |
| OPERATIONS GUIDE                               |                          |      |   |
| Membership Service Providers (MSP)             | msp                      |  309 | - |
| Channel Configuration (configtx)               | configtx                 |  490 | - |
| Channel Configuration (configtxgen)            | configtxgen              |  332 | - |
| Reconfiguring with configtxlator               | configtxlator            |  293 | - |
| Endorsement policies                           | endorsement-policies     |  106 | - |
| Error handling                                 | error-handling           |  151 | - |
| Logging Control                                | logging-control          |  209 | - |
| ARCHITECTURE                                   |                          |      |   |
| Architecture Explained                         | arch-deep-dive           |  831 | - |
| Transaction Flow                               | txflow                   |  117 | DONE |
| Hyperledger Fabric CA's User Guide             |                 -        |    - | - |
| Hyperledger Fabric SDKs                        | fabric-sdks              |   13 | DONE |
| Bringing up a Kafka-based Ordering Service     | kafka                    |   96 | DONE |
| Channels                                       | channels                 |   44 | DONE |
| Ledger                                         | ledger                   |  168 | TODO |
| Read-Write set semantics                       | readwrite                |  154 | TODO |
| Gossip data dissemination protocol             | gossip                   |   87 | DONE |
| TROUBLESHOOTING AND FAQS                       |                          |      |   |
| Hyperledger Fabric FAQ                         | Fabric-FAQ               |  171 | TODO |
| APPENDIX                                       |                          |      |   |
| Glossary                                       | glossary                 |  369 | DONE |

## Introduction ##

This document contains information on how the Fabric documentation is
built and published as well as a few conventions one should be aware of
before making changes to the doc.

The crux of the documentation is written in
[reStructuredText](http://docutils.sourceforge.net/rst.html) which is
converted to HTML using [Sphinx](http://www.sphinx-doc.org/en/stable/).
The HTML is then published on http://hyperledger-fabric.readthedocs.io
which has a hook so that any new content that goes into `docs/source`
on the main repository will trigger a new build and publication of the
doc.

## Conventions ##

* Source files are in RST format and found in the `docs/source` directory.
* The main entry point is index.rst, so to add something into the Table
  of Contents you would simply modify that file in the same manner as
  all of the other topics. It's very self-explanatory once you look at
  it.
* Relative links should be used whenever possible. The preferred
  syntax for this is: :doc:\`anchor text &lt;relativepath&gt;\`
  <br/>Do not put the .rst suffix at the end of the filepath.
* For non RST files, such as text files, MD or YAML files, link to the
  file on github, like this one for instance:
  https://github.com/hyperledger/fabric/blob/master/docs/README.md

Notes: The above means we have a dependency on the github mirror
repository. Relative links are unfortunately not working on github
when browsing through a RST file.

## Setup ##

Making any changes to the documentation will require you to test your
changes by building the doc in a way similar to how it is done for
production. There are two possible setups you can use to do so:
setting up your own staging repo and publication website, or building
the docs on your machine. The following sections cover both options:

### Setting up your own staging repo and publication website ###

You can easily build your own staging repo following these steps:

1. Fork [fabric on github](https://github.com/hyperledger/fabric)
1. From your fork, go to `settings` in the upper right portion of the screen,
1. click `Integration & services`,
1. click `Add service` dropdown,
1. and scroll down to ReadTheDocs.
1. Next, go to http://readthedocs.org and sign up for an account. One of the first prompts will offer to link to github. Elect this then,
1. click import a project,
1. navigate through the options to your fork (e.g. yourgithubid/fabric),
1. it will ask for a name for this project. Choose something

intuitive. Your name will preface the URL and you may want to append `-fabric` to ensure that you can distinguish between this and other docs that you need to create for other projects. So for example:
`yourgithubid-fabric.readthedocs.io/en/latest`

Now anytime you modify or add documentation content to your fork, this
URL will automatically get updated with your changes!

### Building the docs on your machine ###

Here are the quick steps to achieve this on a local machine without
depending on ReadTheDocs, starting from the main fabric
directory. Note: you may need to adjust depending on your OS.

``` shell
sudo pip install Sphinx
sudo pip install sphinx_rtd_theme
cd fabric/docs # Be in this directory. Makefile sits there.
make html
```

This will generate all the html files in `docs/build/html` which you can
then start browsing locally using your browser. Every time you make a
change to the documentation you will of course need to rerun `make
html`.

In addition, if you'd like, you may also run a local web server with the following commands (or equivalent depending on your OS):

``` shell
sudo apt-get install apache2
cd source/build/html
sudo cp -r * /var/www/html/
```

You can then access the html files at `http://localhost/index.html`.

[![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "Creative Commons License")](http://creativecommons.org/licenses/by/4.0/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).
