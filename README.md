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

Este guião é inteiramente realizado através do **terminal**. Para quem nunca o utilizou:

- No **Windows**, deve abrir-se o PowerShell (Iniciar → escrever “PowerShell” → Enter)
- No **macOS**, deve abrir-se a aplicação Terminal (Aplicações > Utilitários > Terminal)

### Windows
```bash
# Em primeiro lugar, descarrega-se e instala-se o GPG4Win
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
Se for apresentado algo como `gpg (GnuPG) 2.x.x`, está tudo bem.

> A instalação pode demorar alguns minutos, dependendo da ligação à internet ou do sistema em utilização.

---

## Parte 2: Criar o par de chaves

```bash
gpg --full-generate-key
```

### Explicação passo-a-passo

Ao correr o comando, será apresentada uma série de perguntas no terminal. Em cada uma, deve ser introduzido um número ou um valor, seguido de `Enter`. Abaixo explicam-se cada um dos passos.
1. **Tipo de chave**: Deve escolher-se `1` para RSA e RSA — é o mais compatível e permite tanto cifrar como assinar.
2. **Tamanho**: Pode aceitar-se o valor por defeito (`3072` bits) ou escrever `4096` para mais segurança (embora seja um pouco mais lento).
3. **Validade**: Para este laboratório, deve escrever-se `1m` (um mês).
4. **Nome real**: O nome da pessoa (ex: Alice)
5. **Email**: O email institucional ou pessoal
6. **Comentário**: Campo opcional (pode deixar-se em branco)
7. **Frase secreta (passphrase)**: Protege a chave privada. Deve ser forte. Durante a escrita, os caracteres não serão visíveis no ecrã (por segurança). Podem usar-se as setas para cima/baixo para repetir a mesma frase em ambos os campos (entrada e confirmação), se necessário.

### Verificar a chave criada
```bash
gpg --list-keys
```
Ver-se-á algo como:
```
pub   rsa3072 2025-06-12 [SC] [expira: 2025-07-12]
      FC37899E793EE7FEB60C528DBC603AA4E2C3BE5D
uid         [   plena  ] Alice <alice@tecnico.ulisboa.pt>
sub   rsa3072 2025-06-12 [E] [expira: 2025-07-12]
```

### Como interpretar isto:
- `pub`: chave pública principal (usada para **assinar** e **cifrar**)
- `sub`: subchave (gerada automaticamente, usada normalmente só para **cifrar**)
- `uid`: identificação da chave (nome + email)

> Este é o “cartão de cidadão criptográfico”.

---

## Parte 3: Cifrar e Decifrar Mensagens

### Criar um ficheiro simples
```bash
echo "Este ficheiro é secreto." > segredo.txt
```

### Cifrar para si próprio (usando a chave pública)
```bash
gpg --encrypt --recipient alice@tecnico.ulisboa.pt segredo.txt
```

### Explicação
- `--encrypt`: indica que se pretende cifrar
- `--recipient`: define o destinatário da mensagem (usa-se o email associado à chave)
- Será criado o ficheiro `segredo.txt.gpg` (ficheiro cifrado)

### Tentar abrir o ficheiro cifrado
```bash
cat segredo.txt.gpg
```
Deverá ver-se um **bloco de texto ilegível** — mesmo quem cifrou não o consegue ler diretamente.

Exemplo de saída:
```
����[��KѪ�G���Κx%�tߞc�aМdO��9��0?�sU|2!3גesm~�E?w��h/V?r�u�n�PFy2:?&y<g{$?$!uQ2?A= �f~beY?]?'?n...
```

### Decifrar (com a chave privada)
```bash
gpg --decrypt segredo.txt.gpg
```
Ao inserir a frase secreta, será possível ver o conteúdo em texto legível.

---

## Parte 4: Assinar e Verificar Ficheiros

### Objetivo
Quando se cria um ficheiro e se pretende enviá-lo a alguém, como pode essa pessoa ter a certeza de que foi mesmo o autor quem o escreveu — e de que ninguém o alterou durante a transmissão?

A assinatura digital serve precisamente para esse propósito:
- **Garante a autoria** (foi, de facto, a pessoa detentora da chave privada que assinou)
- **Garante a integridade** (o conteúdo não foi modificado)

No PGP, assinar um ficheiro envolve usar a **chave privada** para gerar um código único que só pode ser verificado com a **chave pública** correspondente. É como "selar" digitalmente um documento com a tua identidade.

### Assinar
```bash
gpg --clearsign segredo.txt
```

Será gerado o ficheiro `segredo.txt.asc`, contendo o conteúdo original e a assinatura digital.

### Verificar assinatura
```bash
gpg --verify segredo.txt.asc
```
Se o ficheiro não tiver sido alterado, a assinatura será considerada válida.

---

## Parte 5: Trocar Mensagens com Colegas

Uma vez criada a chave, é o momento de proceder a uma experiência prática com outra pessoa. Recomenda-se a colaboração com um colega (ou, em alternativa, a utilização de dois terminais distintos) para a realização dos passos seguintes.

### 1. Exportar a chave pública

Windows:
```bash
gpg --export --armor alice@tecnico.ulisboa.pt | Out-File -Encoding ascii -FilePath chave_publica.asc
```

Linux/MacOS:
```bash
gpg --export --armor alice@tecnico.ulisboa.pt > chave_publica.asc
```

Explicação:
- Cria-se um ficheiro `chave_publica.asc` com a chave pública, em formato de texto, codificado com ASCII, em vez de UTF-16, que é o padrão da powershell. (IMPORTANTE para o --import seguinte)
- Este ficheiro é seguro para partilhar — pode ser entregue ao colega.

### 2. Importar a chave pública de outra pessoa
```bash
gpg --import chave_bob.asc
```

Explicação:
- Isto adiciona a chave pública do colega ao sistema.
- A partir deste momento, **podem ser enviadas mensagens cifradas que só essa pessoa conseguirá ler**.

### 3. Cifrar uma mensagem para o colega
```bash
echo "Ola Bob" > mensagem.txt
gpg --encrypt --recipient bob@tecnico.ulisboa.pt mensagem.txt
```
O valor de `--recipient` deve ser o email que o colega utilizou ao criar o seu par de chaves — é assim que o GPG identifica a chave pública correspondente para cifrar a mensagem.

### 4. (Opcional) Cifrar + Assinar
```bash
gpg --encrypt --sign --recipient bob@tecnico.ulisboa.pt mensagem.txt
```

Explicação:
- `--sign`: assinas com a chave privada
- Garante que o destinatário pode confirmar que foi o remetente legítimo que enviou a mensagem

> Este ficheiro (`.gpg`) é o que deve ser enviado ao colega — por email, pen ou outro meio. A pessoa destinatária conseguirá abri-lo e verificar quem o enviou, desde que tenha a chave pública do remetente importada. Lembra-se do comando para decifrar?
> 
> **Na prática, é comum cifrar + assinar**: desta forma, protege-se o conteúdo e confirma-se a identidade do remetente.

---

## Parte 6: Servidores de Chaves Públicas (Exercício)

### Objectivo:

Publicar uma chave num servidor global, de forma a que possa ser encontrada por qualquer pessoa.

### Instruções

1. Criar uma chave PGP com o **primeiro nome e apelido** (ex: `Joana Silva`), ou utilizar a que foi criada anteriormente.
2. Descobre o ID da chave com:

   ```bash
   gpg --list-keys
   ```

3. Envia a chave para o servidor global:

   ```bash
   gpg --send-keys ABCD1234EF567890
   ```

   (Deve substituir-se `ABCD1234EF567890` pelo **ID da chave**, visível na linha `pub`)

4. Aceder a [https://keys.openpgp.org](https://keys.openpgp.org) e procurar a chave pelo nome ou email.

Deverá ser possível vê-la publicada online.

---

## Parte 7: Revogação de Chaves (Exercício)

### Objectivo:

Simular uma situação de perda ou compromisso da chave privada, criando e utilizando um certificado de revogação.

### Instruções

1. Criar um certificado de revogação para a chave anteriormente gerada:

     ```bash
   gpg --gen-revoke alice@tecnico.ulisboa.pt > revogacao.asc
   ```
     
2. Importar o certificado para revogar a chave:

   ```bash
   gpg --import revogacao.asc
   ```

3. Actualizar essa informação nos servidores:

   ```bash
   gpg --send-keys ABCD1234EF567890
   ```

4. Aceder novamente a https://keys.openpgp.org e confirmar que a chave aparece como revogada.

---

## Parte 8: Criar uma nova chave (continuação do exercício)

### Instruções

1. Criar uma nova chave PGP, utilizando o mesmo nome da anterior mas acrescentando o sufixo `_2025` (ex: `Joana Silva_2025`)

2. Enviar a nova chave para o servidor global:

   ```bash
   gpg --send-keys NOVO_ID_DA_CHAVE
   ```

3. Confirmar, através de https://keys.openpgp.org, que a nova chave foi publicada com sucesso.
> Este exercício demonstra não apenas como publicar uma chave, mas também como revogá-la e substituí-la — uma prática essencial na gestão segura de identidades criptográficas.

---

## Parte 9: Backup e Eliminação de Chaves (Opcional)

### Exportar chave privada (cuidado!)
```bash
gpg --export-secret-keys --armor alice@tecnico.ulisboa.pt > minha_privada.asc
```
A chave privada deve ser guardada num local seguro, como um disco encriptado ou uma pen USB protegida.

### Eliminar chaves localmente
```bash
gpg --delete-secret-keys alice@tecnico.ulisboa.pt
gpg --delete-key alice@tecnico.ulisboa.pt
```

---

## Parte 10: Uso do PGP no Email

### Objectivo

Compreender como o PGP pode ser utilizado no envio de mensagens seguras por correio electrónico.

### Email seguro

O PGP permite:

- **Cifrar emails** → só o destinatário pode ler
- **Assinar digitalmente** → o destinatário confirma quem enviou e que o conteúdo não foi alterado

### Exemplos de clientes compatíveis

- **Thunderbird** (com suporte PGP integrado)
- **ProtonMail** (suporte PGP nativo e fácil de usar)

O uso do PGP no email é comum em contextos académicos, profissionais e de activismo digital, sempre que se pretende proteger comunicações sensíveis.


---

## Perguntas para Reflexão
1. Em que situações se poderia utilizar o PGP na vida pessoal ou profissional?
2. Que limitações apresenta o PGP enquanto sistema de segurança?
3. Como se pode confirmar que uma determinada chave pública é autêntica?
4. O que acontece caso se perca o acesso à chave privada?

---

## Projeto Opcional em Python

Deve utilizar-se a biblioteca `python-gnupg` para desenvolver um programa que permita a troca de mensagens seguras.

```python
import gnupg

gpg = gnupg.GPG()
gpg.encoding = 'utf-8'

with open('mensagem.txt', 'rb') as f:
    encrypted = gpg.encrypt_file(
        f,
        recipients=['bob@tecnico.ulisboa.pt'],
        sign='alice@tecnico.ulisboa.pt',
        passphrase='FraseSecretaAqui',
        output='mensagem.txt.gpg'
    )
print('Cifrado com sucesso?', encrypted.ok)
```

---

## Tarefa Final Sugerida

> Redigir um pequeno texto (3 a 5 linhas) descrevendo um cenário profissional ou académico em que o uso de PGP seja considerado útil. O texto deve incluir:
>
> - Um benefício concreto
> - Uma limitação ou dificuldade

---

## Conclusão

Com o estudo e prática do PGP, foram desenvolvidas competências fundamentais na área da segurança da informação, nomeadamente:
- Criação e gestão de pares de chaves
- Cifra e decifra de ficheiros
- Assinatura digital e verificação de integridade
- Troca segura de chaves públicas
- Gestão de revogação e cópias de segurança de chaves

PGP é uma ferramenta poderosa — mas só é eficaz se **usada corretamente** e com cuidado.

> "A criptografia é como um cofre: se perderes a chave ou deixares a porta aberta, não vale de nada ser de aço reforçado."
