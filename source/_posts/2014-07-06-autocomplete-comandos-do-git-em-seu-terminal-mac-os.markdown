---
layout: post
title: "Autocomplete comandos do Git em seu terminal Mac OS"
date: 2014-07-06 16:19:12 -0300
comments: true
categories: [git, mac os, desenvolvimento]
---

No termina/shell do Mac OS X você pode usar [TAB] para autocompletar caminhos de arquivos e diretórios. Já imaginou como seria incrível se pudesse usar essa mesma função para comandos git assim como nome de branches?

Então você pode!!! Veja como.

O primeiro passo é baixar o script `git-completion.bash` (veja [aqui](https://github.com/git/git/blob/master/contrib/completion/git-completion.bash)) em seu diretório raiz executando o seguinte comando:

{% codeblock lang:bash %}
curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -o ~/.git-completion.bash
{% endcodeblock %}

O próximo passo é adicionar o código abaixo em seu arquivo `~/.bash_profile`. 
Este trecho de código executa o script git autocomplete caso ele exista.

{% codeblock lang:bash %}
if [ -f ~/.git-completion.bash ]; then
  . ~/.git-completion.bash
fi
{% endcodeblock %}

Agora abra um novo shell, vá para um diretório que contenha um repositório git e comece a digitar um comando. Agora você pode usar [TAB] para autocompletar comandos do git e nomes de branch. 

Por exemplo, se você digitar git + espaço + [TAB], você vai ter como resultado uma lista contendo todos os comandos disponíveis do git:

{% codeblock lang:bash %}
add                      config                   instaweb                 reset 
am                       credential               log                      revert 
annotate                 credential-osxkeychain   merge                    rm 
apply                    describe                 mergetool                send-email 
archive                  diff                     mv                       shortlog 
bisect                   difftool                 name-rev                 show 
blame                    fetch                    notes                    show-branch 
branch                   filter-branch            p4                       stage 
bundle                   format-patch             pull                     stash 
checkout                 fsck                     push                     status 
cherry                   gc                       rebase                   submodule 
cherry-pick              get-tar-commit-id        reflog                   subtree 
citool                   grep                     relink                   svn 
clean                    gui                      remote                   tag 
clone                    help                     repack                   whatchanged 
column                   imap-send                replace                  
commit                   init                     request-pull   
{% endcodeblock %}

A brincadeira fica ainda mais interessante quando você trabalha com várias branches. O processo de checkout e push fica bem mais rápido!!! Bom, isso para quem gosta de linhas de comandos ;)
