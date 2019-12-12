---
layout: blogger
title: GPG Keys para e-mail, assinatura digital, criptografia e Yubikey no Windows
published: false
tags: [Segurança, Criptografia, GPG]
---
https://suchsecurity.com/gpg-and-ssh-with-yubikey-on-windows.html
https://www.hanselman.com/blog/HowToSetupSignedGitCommitsWithAYubiKeyNEOAndGPGAndKeybaseOnWindows.aspx
https://www.esev.com/blog/post/2015-01-pgp-ssh-key-on-yubikey-neo/

https://keybase.io/

# Criar chave master:

- Baixar e instalar GPG para Windows no link [https://www.gpg4win.org/]
- Executar o comando ```gpg -- expert --full-generate-key```
- Escolher a opção 8: RSA (set your own capabilities)
- Desabilitar assinatura e criptografia, para que a chave master só possa ser usada para criar sub chaves. 
- Escolha 4096 para a chave master
- Escolha o tempo de validade da chave: *configurar a chave para expirar no dia do aniversário a cada 2 ou 3 anos*
- Preencher os dados pessoais
- Definir senha
- Salvar os arquivos gerados da chave privada e chave de revocação

## Adicionar e-mails na chave master

- Executar gpg ```--edit-key <fingerprint>``` depois o comando ```adduid```
- Preencher o nome e novo e-mail
- Digitar a senha da chave privada

## Criar subchaves:

- Executar gpg ```--edit-key <fingerprint>``` depois o comando ```addkey```
- Escolher a opçao 6: RSA (encrypt only / apenas cifragem)
- Escolher tamanho 4096
- Escolher tempo de validade
