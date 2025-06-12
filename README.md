# Guião de Laboratório: Introdução ao PGP (Pretty Good Privacy)
**Instituto Superior Técnico, Universidade de Lisboa**  
**Fundamentos da Segurança**  

---

## Introdução

Vivemos numa era em que os nossos dados circulam por servidores, aplicações e redes em todo o mundo. Como podemos garantir que:

- Só o destinatário certo consegue ler uma mensagem?
- O conteúdo não foi alterado durante a transmissão?
- A pessoa que enviou é mesmo quem diz ser?

É aqui que entra o **PGP – Pretty Good Privacy**, um sistema de criptografia que nos permite proteger mensagens e ficheiros mesmo em ambientes não seguros.

---

## Cifras Simétricas vs Cifras Assimétricas

### Cifra Simétrica
- Uma **única chave secreta** é usada para **cifrar** (transformar texto legível em incompreensível) e **decifrar** (voltar ao original).
- Rápido e simples.
- Problema: como enviar essa chave ao destinatário de forma segura?

### Cifra Assimétrica
- Cada pessoa tem um **par de chaves**:
  - Chave **pública**: pode ser partilhada com qualquer pessoa.
  - Chave **privada**: deve ser mantida em segredo absoluto.
- Uma é usada para cifrar, a outra para decifrar — e o mesmo princípio permite também verificar a identidade de quem envia mensagens.

**PGP é um sistema que usa cifra assimétrica para proteger dados e autenticar comunicações.**

---

## O que é o PGP?

PGP (Pretty Good Privacy) é um protocolo de segurança criado para garantir:
- **Confidencialidade**: só quem tiver a chave certa pode ler a mensagem
- **Autenticidade**: sabes quem enviou a mensagem
- **Integridade**: tens a certeza que a mensagem não foi alterada
- **Não-repúdio**: o autor não pode negar que enviou a mensagem

O PGP moderno é implementado por uma ferramenta chamada **GnuPG** (ou GPG).

---

## Parte 1: Instalação e Verificação

Este guião é feito inteiramente usando **o terminal**. Se nunca usaste:

- No **Windows**, abre o PowerShell (Iniciar → escreve “PowerShell” → Enter)
- No **macOS**, abre a aplicação Terminal (Applications > Utilities > Terminal)

### Windows
```bash
# Primeiro, descarrega e instala o GPG4Win:
https://gpg4win.org
```

### macOS
```bash
brew install gnupg
```

### Verificar instalação
```bash
gpg --version
```
Se vires algo como `gpg (GnuPG) 2.x.x`, está tudo bem.

> A instalação pode demorar alguns minutos dependendo da tua internet ou do sistema.

---

## Parte 2: Criar o teu par de chaves

```bash
gpg --full-generate-key
```

### Explicação passo-a-passo

Quando correres o comando, vais ver uma série de perguntas no terminal. Em cada uma, tens de escrever um número ou um valor e carregar em `Enter`. Abaixo explicamos cada passo.
1. **Tipo de chave**: Escolhe `1` para RSA e RSA — é o mais compatível e permite tanto cifrar como assinar.
2. **Tamanho**: Podes aceitar o valor por defeito (`3072` bits) ou escrever `4096` para mais segurança (embora seja um pouco mais lento).
3. **Validade**: Para este laboratório, escreve `1m` (um mês).
4. **Nome real**: O teu nome (ex: João Cripto)
5. **Email**: O teu email institucional ou pessoal
6. **Comentário**: Algo opcional (podes deixar em branco)
7. **Frase secreta (passphrase)**: Protege a tua chave privada. Deve ser forte! Durante a escrita, não vais ver os caracteres no ecrã (por segurança). Podes usar as setas para cima/baixo para repetir a mesma frase em ambos os campos (entrada e confirmação), se precisares.

### Verificar a chave criada
```bash
gpg --list-keys
```
Vais ver algo como:
```
pub   rsa3072 2025-06-12 [SC] [expira: 2025-07-12]
      FC37899E793EE7FEB60C528DBC603AA4E2C3BE5D
uid         [   plena  ] João Cripto <joao@tecnico.ulisboa.pt>
sub   rsa3072 2025-06-12 [E] [expira: 2025-07-12]
```

### Como interpretar isto:
- `pub`: chave pública principal (usada para **assinar** e **cifrar**)
- `sub`: subchave (gerada automaticamente, usada normalmente só para **cifrar**)
- `uid`: identificação da chave (nome + email)

> Este é o teu “cartão de cidadão criptográfico”.

---

## Parte 3: Cifrar e Decifrar Mensagens

### Criar um ficheiro simples
```bash
echo "Este ficheiro é secreto." > segredo.txt
```

### Cifrar para ti próprio (usando tua chave pública)
```bash
gpg --encrypt --recipient joao@tecnico.ulisboa.pt segredo.txt
```

### Explicação
- `--encrypt`: indica que queres cifrar
- `--recipient`: para quem é a mensagem (usa o email da chave)
- Vai criar `segredo.txt.gpg` (ficheiro cifrado)

### Tenta abrir o ficheiro cifrado
```bash
cat segredo.txt.gpg
```
Deverás ver um **bloco de texto ilegível** — mesmo tu não consegues lê-lo diretamente.

Exemplo de saída:
```
����[��KѪ�G���Κx%�tߞc�aМdO��9��0?�sU|2!3גesm~�E?w��h/V?r�u�n�PFy2:?&y<g{$?$!uQ2?A= �f~beY?]?'?n...
```

### Decifrar (com tua chave privada)
```bash
gpg --decrypt segredo.txt.gpg
```
Se inserires a tua frase secreta, vês o conteúdo claro.

---

## Parte 4: Assinar e Verificar Ficheiros

### Objetivo
Quando crias um ficheiro e o queres enviar a alguém, como é que essa pessoa sabe que foste mesmo tu que o escreveste — e que ninguém o alterou pelo caminho?

A assinatura digital serve para isso:
- **Garante a autoria** (foste mesmo tu a assinar)
- **Garante a integridade** (o conteúdo não foi modificado)

No PGP, assinar um ficheiro envolve usar **a tua chave privada** para gerar um código único que só pode ser verificado com a **tua chave pública**. É como "selar" digitalmente um documento com a tua identidade.

### Assinar
```bash
gpg --clearsign segredo.txt
```

Vai gerar `segredo.txt.asc` com o conteúdo + assinatura.

### Verificar assinatura
```bash
gpg --verify segredo.txt.asc
```
Se ninguém mexeu no ficheiro, a assinatura será válida.

---

## Parte 5: Trocar Mensagens com Colegas

Agora que já tens a tua chave, está na hora de **experimentar com outra pessoa**. Combina com um colega (ou usa dois terminais diferentes) e façam os seguintes passos:

### 1. Exportar a tua chave pública
```bash
gpg --export --armor joao@tecnico.ulisboa.pt > chave_publica.asc
```

Explicação:
- Cria um ficheiro `chave_publica.asc` com a tua chave pública, em formato de texto.
- Este ficheiro é seguro para partilhar — dá-o ao teu colega!

### 2. Importar a chave pública de outra pessoa
```bash
gpg --import chave_pachi.asc
```

Explicação:
- Isto adiciona a chave pública do teu colega ao teu sistema.
- A partir deste momento, **podes enviar mensagens cifradas só ele/a conseguirá ler**.

### 3. Cifrar uma mensagem para o teu colega
```bash
echo "Ola Pachi" > mensagem.txt
gpg --encrypt --recipient pachicodes@dev.to mensagem.txt
```

### 4. (Opcional) Cifrar + Assinar
```bash
gpg --encrypt --sign --recipient pachicodes@dev.to mensagem.txt
```

Explicação:
- `--sign`: assinas com a tua chave privada
- Isso garante que o destinatário sabe que foste **tu** a enviar

> Este ficheiro (`.gpg`) é o que deves enviar ao teu colega — por email, pen, ou outro meio. Ele(a) conseguirá abrir e verificar quem o enviou, desde que tenha a tua chave pública importada. Lembras-te do comando para decifrar?
> 
> **Na prática, é comum cifrar + assinar**: assim proteges o conteúdo **e** confirmas a identidade do remetente.

---

## Parte 6: Servidores de Chaves Públicas

### Enviar a tua chave para um servidor global
```bash
gpg --send-keys ABCD1234EF567890
```
(Substitui `ABCD1234EF567890` pelo **ID da tua chave**, que podes ver com `gpg --list-keys` na linha abaixo de `pub`)

### Para quê?

Enviar a tua chave para um servidor global permite que **qualquer pessoa no mundo** a possa encontrar e usar para te enviar mensagens seguras.

Isto é especialmente útil quando:
- Estás a colaborar remotamente com alguém
- Vais publicar a tua chave no site pessoal ou email institucional
- Queres que te possam contactar de forma segura sem teres de enviar manualmente o ficheiro `.asc`

### Exemplos de servidores onde podes procurar e partilhar chaves:
- https://keys.openpgp.org
- https://keyserver.ubuntu.com

> Depois de enviada, podes procurar a tua chave pelo nome ou email nesses sites.

---

## Parte 7: Revogação de Chaves

### Para quê?

E se perderes o acesso à tua chave privada? Ou se alguém a conseguir roubar? Neste caso, deves **revogar** a tua chave — ou seja, declarar publicamente que ela já não é válida.

Para isso, usa um **certificado de revogação**, que serve como um "cancelamento oficial" da tua identidade digital associada a essa chave.

> **É importante criar este certificado logo após criar a chave!** Guarda-o num sítio seguro (ex: pen USB offline).

---

### Criar certificado de revogação
```bash
gpg --gen-revoke joao@tecnico.ulisboa.pt > revogacao.asc
```

- Vai gerar um ficheiro `revogacao.asc`
- Guarda este ficheiro num local seguro, como backup de emergência

---

### Usar o certificado (caso a chave esteja comprometida)
```bash
gpg --import revogacao.asc
gpg --send-keys ABCD1234EF567890
```

- Isto marca a chave como revogada no teu sistema
- O comando `--send-keys` atualiza essa informação nos servidores de chaves públicos

Depois disso, outras pessoas verão que a tua chave **já não deve ser usada**

---

## Parte 8: Backup e Eliminação de Chaves

### Exportar chave privada (cuidado!)
```bash
gpg --export-secret-keys --armor joao@tecnico.ulisboa.pt > minha_privada.asc
```
Guardar em disco encriptado ou pen USB segura.

### (Opcional) Apagar chaves localmente
```bash
gpg --delete-secret-keys joao@tecnico.ulisboa.pt
gpg --delete-key joao@tecnico.ulisboa.pt
```

---

## Parte 9: Casos de Uso Reais

### Email Seguro
- Enviar emails cifrados e assinados
- Thunderbird + Enigmail ou ProtonMail

### Software
- Assinatura de programas distribuídos (evitar malware)

### Blockchain
- Assinaturas digitais são a base de transações em Bitcoin, Ethereum, etc.

---

## Perguntas para Reflexão

1. Quando usarias PGP na tua vida pessoal ou profissional?
2. Que limitações tem o PGP?
3. Como confirmarias que a chave pública de alguém é autêntica?
4. O que acontece se perderes a tua chave privada?

---

## Projeto Sugerido em Python

Usa `python-gnupg` para criar um programa de troca de mensagens seguras:

```python
import gnupg

gpg = gnupg.GPG()
gpg.encoding = 'utf-8'

with open('mensagem.txt', 'rb') as f:
    encrypted = gpg.encrypt_file(
        f,
        recipients=['pachi@example.com'],
        sign='joao@tecnico.ulisboa.pt',
        passphrase='FraseSecretaAqui',
        output='mensagem.txt.gpg'
    )
print('Cifrado com sucesso?', encrypted.ok)
```

---

## Tarefa Final Sugerida

> Escreve um pequeno texto (1 página máx.) onde descreves **um cenário profissional ou académico** onde consideres útil o uso de PGP. Inclui:
>
> - Um benefício concreto
> - Uma limitação ou dificuldade

---

## Conclusão

Com o PGP aprendeste a:
- Criar e gerir pares de chaves
- Cifrar e decifrar ficheiros
- Assinar digitalmente e verificar integridade
- Trocar chaves de forma segura
- Lidar com revogação e backup de chaves

PGP é uma ferramenta poderosa — mas só é eficaz se **usada corretamente** e com cuidado.

> "A criptografia é como um cofre: se perderes a chave ou deixares a porta aberta, não vale de nada ser de aço reforçado."
