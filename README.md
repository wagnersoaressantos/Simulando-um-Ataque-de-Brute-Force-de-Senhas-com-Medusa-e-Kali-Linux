# 🛡️ Desafio de Segurança Cibernética do bootcamp Riachuelo - Cibersegurança: Força Bruta com Medusa no Kali Linux

Este repositório documenta a execução de um desafio prático focado na simulação de ataques de força bruta contra serviços de rede comuns. O objetivo foi exercitar a configuração de laboratório, reconhecimento de alvos, enumeração de usuários e a exploração de credenciais fracas utilizando ferramentas nativas do Kali Linux, como Nmap, Enum4Linux e Medusa, em um ambiente controlado contra a máquina vulnerável Metasploitable 2.

---

## 📖 Visão Geral do Projeto

Este projeto demonstra profissionalmente o ciclo de um ataque de força bruta, partindo do zero até a obtenção de acesso válido. O foco não foi apenas o ataque, mas o entendimento técnico dos protocolos e a validação das descobertas.

---

## 🛠️ Ferramentas Utilizadas

* **Atacante:** Kali Linux (Rolling Edition)
* **Alvo (Vulnerable VM):** Metasploitable 2
* **Reconhecimento:** Nmap
* **Enumeração SMB:** Enum4Linux
* **Ferramenta de Ataque (Brute Force):** Medusa
* **Virtualização:** Oracle VirtualBox (Rede Host-only)

---

## 🌐 1. Configuração do Ambiente de Rede

A primeira etapa foi garantir o isolamento das máquinas em uma rede Host-only no VirtualBox para assegurar que nenhum tráfego de teste saísse para a internet.

* **Kali Linux:** Configurou a interface `eth0` com o IP `192.168.56.102/24`.

<p align="center">
  <img src="./img/ipKali.png" width="700px" alt="Configuração de IP do Kali Linux">
  <br><em>Figura 1: Validação do endereço IP da máquina atacante via terminal do Kali.</em>
</p>

* **Metasploitable 2:** A máquina alvo recebeu o IP `192.168.56.101/24` na interface `eth0`.

<p align="center">
  <img src="./img/ipmsf.png" width="700px" alt="Configuração de IP do Metasploitable">
  <br><em>Figura 2: Validação do endereço IP da máquina alvo Metasploitable 2.</em>
</p>

---

## 🔍 2. Reconhecimento e Exploração

Com o laboratório montado, iniciou-se o ciclo de Pentest.

### Fase 2a: Reconhecimento com Nmap

Utilizou-se o Nmap para descobrir quais portas e serviços estavam abertos no alvo. O comando especificou as portas mais comuns para ataques de força bruta: 21 (FTP), 22 (SSH), 80 (HTTP), 139 e 445 (SMB).

* **Comando:** `nmap -sV -p 21,22,80,445,139 192.168.56.101`

<p align="center">
  <img src="./img/nmap.png" width="700px" alt="Resultado do Scan do Nmap">
  <br><em>Figura 3: Resultado do Nmap mostrando os serviços FTP (vsftpd 2.3.4) e SMB (Samba) abertos.</em>
</p>

### Fase 2b: Enumeração SMB com Enum4Linux

Sabendo que o SMB estava rodando, utilizou-se o **Enum4Linux** para extrair informações mais detalhadas sobre o alvo, incluindo a estrutura do sistema operacional e, crucialmente, uma lista de usuários válidos.

<p align="center">
  <img src="./img/usandoEnum.png" width="700px" alt="Início do Enum4Linux">
  <br><em>Figura 4: Início da execução do Enum4Linux contra o IP 192.168.56.101.</em>
</p>

<p align="center">
  <img src="./img/usuariosEncontrados.png" width="700px" alt="Lista de usuários extraída pelo Enum4Linux">
  <br><em>Figura 5: Lista de usuários extraídos do alvo, incluindo 'nobody', 'root', e 'msfadmin'. Essa lista é fundamental para criar wordlists personalizadas e eficazes.</em>
</p>

---

## 🚀 3. Execução dos Ataques de Força Bruta

Com usuários e serviços identificados, preparou-se os ataques de força bruta utilizando o **Medusa**.

### Cenário 3a: Ataque de Credenciais Cruzadas no FTP

Neste cenário, utilizou-se uma lista de usuários (`users.txt`) contra uma lista de senhas (`pass.txt`) no protocolo FTP. O Medusa testou todas as combinações cruzadas.

* **Comando:** `medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6`
* **Observação:** O parâmetro `-t 6` foi usado para paralelismo moderado, evitando sobrecarregar o serviço.

<p align="center">
  <img src="./img/ataqueftp.png" width="700px" alt="Ataque FTP bem-sucedido com Medusa">
  <br><em>Figura 6: O Medusa identificou credenciais válidas. Notavelmente, 'msfadmin'/'msfadmin', um exemplo claro de credenciais padrão fracas.</em>
</p>

### Cenário 3b: Password Spraying no SMB (Samba)

Neste teste, utilizou-se a técnica de Password Spraying, testando uma única senha fraca contra múltiplos usuários.

* **Comando:** `medusa -h 192.168.56.101 -U msfuser.txt -P msfpass.txt -M smbnt -t 2 -t 50`

<p align="center">
  <img src="./img/smbnt.png" width="700px" alt="Ataque SMB bem-sucedido com Medusa">
  <br><em>Figura 7: O Medusa validou credenciais via SMB.</em>
</p>

---

## ✅ 4. Validação de Acesso

O sucesso do teste não está apenas no terminal do Medusa, mas na prova de que o acesso é funcional.

### Validação via FTP

Para confirmar, logou-se manualmente no serviço FTP utilizando as credenciais 'msfadmin'/'msfadmin' encontradas pelo Medusa.

<p align="center">
  <img src="./img/loginftp.png" width="700px" alt="Acesso FTP manual bem-sucedido">
  <br><em>Figura 8: Login FTP manual bem-sucedido, provando a eficácia do ataque.</em>
</p>

---

## 🛡️ Medidas de Mitigação Recomendadas

Com base nos resultados, as seguintes medidas defensivas são cruciais:

1.  **Políticas de Senhas Fortes:** Implementar a exigência de senhas complexas (mínimo de 12 caracteres, com letras maiúsculas/minúsculas, números e símbolos) para todos os serviços (FTP, SMB, SSH). Credenciais padrão (como msfadmin) devem ser desativadas ou alteradas imediatamente após a instalação.
2.  **MFA (Autenticação de Dois Fatores):** Esta é a medida mais eficaz. Mesmo que a senha seja quebrada, o atacante não conseguiria acessar sem o segundo fator.
3.  **Bloqueio de Conta (Account Lockout):** Configurar os serviços para bloquear temporariamente contas após um número definido de tentativas de login falhas (ex: 5 tentativas erradas bloqueiam por 30 minutos). Isso impede ataques automatizados.
4.  **Desativação ou Endurecimento (Hardening) do FTP:** O protocolo FTP transmite credenciais em texto claro. Se possível, deve ser desativado ou substituído pelo SFTP (que usa SSH/encriptação). Se necessário, deve ser configurado com listas de IPs permitidos (ACLs).

5.  ## ⚠️ Desafios Técnicos: Automação Web (DVWA)
Durante o desafio, foi realizada uma tentativa de ataque de força bruta contra o formulário de login do **DVWA**. No entanto, a ferramenta Medusa apresentou instabilidades e não conseguiu validar as credenciais corretamente, mesmo com a segurança da aplicação configurada no nível "Low".

**Análise do Problema:**
* **Gestão de Sessão:** Formulários Web modernos (mesmo os deliberadamente vulneráveis) dependem de Cookies de sessão e, por vezes, tokens de segurança que o Medusa, por ser uma ferramenta de força bruta de protocolos mais "estáticos" (como FTP/SMB), pode ter dificuldade em processar de forma síncrona.
* **Complexidade do Módulo HTTP:** O módulo `http` do Medusa exige uma sintaxe muito específica para o envio de parâmetros POST, e qualquer pequena atualização na estrutura do DOM do DVWA pode causar falhas na identificação da string de erro/sucesso.

---

## 🧠 Conclusão e Reflexões Finais

A execução deste desafio proporcionou uma visão realista de um cenário de *Pentest*. Mais do que apenas rodar comandos, o processo exigiu análise crítica e adaptação.

**Principais Aprendizados:**
1. **Ferramenta Certa para o Trabalho Certo:** O Medusa demonstrou uma performance excepcional e extrema rapidez em protocolos de rede como **FTP** e **SMB**. No entanto, ao lidar com a camada de aplicação (Web), ficou claro que ferramentas especializadas em interceptação e automação HTTP (como o **Burp Suite** ou **OWASP ZAP**) seriam mais resilientes para lidar com variáveis como cookies e sessões.
2. **A Importância da Enumeração:** O sucesso dos ataques de força bruta foi diretamente proporcional à qualidade da fase de reconhecimento. O uso do **Enum4Linux** para extrair nomes reais de usuários reduziu drasticamente o "ruído" do ataque e o tempo de execução.
3. **Resiliência e Troubleshooting:** A dificuldade encontrada no ataque ao DVWA foi um dos pontos mais produtivos do desafio. Entender que atualizações de segurança ou mudanças na estrutura da aplicação podem quebrar scripts de automação é fundamental para qualquer profissional de segurança. Isso reforça que o *Pentest* não é um processo mecânico, mas um ciclo constante de tentativa, erro e ajuste técnico.

Este projeto reforça a necessidade de uma defesa em camadas: não basta ter senhas fortes se o protocolo (FTP) é inseguro, e não basta ter um protocolo seguro se as credenciais padrão não forem alteradas.
---

### 👨‍💻 Autor

Desenvolvido por **Wagner Soares**.
* **Formação:** Estudante de Bacharelado em Sistemas de Informação.
* **Interesses:** Python, Data Science, Flutter e Segurança Cibernética.

---
