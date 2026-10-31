# Login Bruteforcing (CWES)

- **Tipos de Bruteforce**
    
    
    | **Method** | **Description** | **Example** | **Best Used When...** |
    | --- | --- | --- | --- |
    | **`Simple Brute Force`** | Systematically tries all possible combinations of characters within a defined character set and length range. | Trying all combinations of lowercase letters from 'a' to 'z' for passwords of length 4 to 6. | No prior information about the password is available, and computational resources are abundant. |
    | **`Dictionary Attack`** | Uses a pre-compiled list of common words, phrases, and passwords. | Trying passwords from a list like 'rockyou.txt' against a login form. | The target will likely use a weak or easily guessable password based on common patterns. |
    | **`Hybrid Attack`** | Combines elements of simple brute force and dictionary attacks, often appending or prepending characters to dictionary words. | Adding numbers or special characters to the end of words from a dictionary list. | The target might use a slightly modified version of a common password. |
    | **`Credential Stuffing`** | Leverages leaked credentials from one service to attempt access to other services, assuming users reuse passwords. | Using a list of usernames and passwords leaked from a data breach to try logging into various online accounts. | A large set of leaked credentials is available, and the target is suspected of reusing passwords across multiple services. |
    | **`Password Spraying`** | Attempts a small set of commonly used passwords against a large number of usernames. | Trying passwords like 'password123' or 'qwerty' against all usernames in an organization. | Account lockout policies are in place, and the attacker aims to avoid detection by spreading attempts across multiple accounts. |
    | **`Rainbow Table Attack`** | Uses pre-computed tables of password hashes to reverse hashes and recover plaintext passwords quickly. | Pre-computing hashes for all possible passwords of a certain length and character set, then comparing captured hashes against the table to find matches. | A large number of password hashes need to be cracked, and storage space for the rainbow tables is available. |
    | **`Reverse Brute Force`** | Targets a single password against multiple usernames, often used in conjunction with credential stuffing attacks. | Using a leaked password from one service to try logging into multiple accounts with different usernames. | A strong suspicion exists that a particular password is being reused across multiple accounts. |
    | **`Distributed Brute Force`** | Distributes the brute forcing workload across multiple computers or devices to accelerate the process. | Using a cluster of computers to perform a brute-force attack significantly increases the number of combinations that can be tried per second. | The target password or key is highly complex, and a single machine lacks the computational power to crack it within a reasonable timeframe. |

---

- **Customizar Wordlist**
    - A partir de apenas o Nome do funcionario, podemos criar wordlists personalizadas para possiveis nomes de usuario e senha.
    - **1. Username**
        - Usamos **`username-anarchy`:**
            
            ```bash
            ./username-anarchy Jane Smith > jane_smith_usernames.txt
            ```
            
    - **2. Password**
        - Usamos **CUPP**. Podemos pesquisar por informacoes sobre o funcionario em redes sociais e juntar elas (Ex: nome de Pets, data de nascimento, etc). Tambem podemos criar a wordlist com um Custom.rule que a empresa tem na Password Policy (ou depois de fazermos a worldlist usamos o **grep** e filtramos).
            
            ```bash
            sudo apt install cupp
            
            cupp -i
            # Agora preenchemos as info, se nao soubermos eh so dar Enter.
            ```
            
        - Agora podemos pegar a wordlist gigante e filtrar **de acordo com a regra de senha da empresa que estamos atacando**, Ex: filtrar a wordlist e pegar apenas as senhas q tem no minimo 8caracteres.
            
            ```bash
            # Pegar apenas as que tem no Minimo 8 caracteres:
            grep -E '^.{8,}$' darkweb2017-top10000.txt > darkweb2017-minlength.txt
            # Pegar apenas as que tem no Minimo 1 letra Maiuscula:
            grep -E '[A-Z]' darkweb2017-minlength.txt > darkweb2017-uppercase.txt
            # Pegar apenas as que tem no Minimo 1 letra Minuscula:
            grep -E '[a-z]' darkweb2017-uppercase.txt > darkweb2017-lowercase.txt
            # Pegar apenas as que tem no minimo 1 numero:
            grep -E '[0-9]' darkweb2017-lowercase.txt > darkweb2017-number.txt
            # Pegar apenas as que tem no minimo 2 caracteres especiais:
            grep -E '([!@#$%^&*].*){2,}' darkweb2017-lowercase.txt > darkweb2017-number.txt
            ```
            
- **Hydra (formulario POST tb)**
    - Bruteforce em Login (**formulario em janela sobreposta** que aparece ao acessar um site (basic auth login))
        
        ```bash
        hydra -l alex -P 2023-200_most_used_passwords.txt 127.0.0.1 http-get / -s 81
        ```
        
    - **Bruteforce em formulario POST:**
        
        ```bash
        hydra -L users.txt -P senhas.txt 10.10.11.123 -s 5000 http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"
        ```
        
        - **`-s 5000`** → porta
        - **`F=Invalid credentials`** → string que o site retorna que indica falha, poderiamos trocar para **`S=`** para indicar a de sucesso.
        - **OBS:** se o site em vez de nos retornar uma string de sucesso/falha no login e retornar um redirecionamento para outra pagina, usamos **`S=<PORTA>`**
- **Medusa**
    
    
    | Medusa Module | Service/Protocol | Description | Usage Example |
    | --- | --- | --- | --- |
    | FTP | File Transfer Protocol | Brute-forcing FTP login credentials, used for file transfers over a network. | `medusa -M ftp -h 192.168.1.100 -u admin -P passwords.txt` |
    | HTTP | Hypertext Transfer Protocol | Brute-forcing login forms on web applications over HTTP (GET/POST). | `medusa -M http -h www.example.com -U users.txt -P passwords.txt -m DIR:/login.php -m FORM:username=^USER^&password=^PASS^` |
    | IMAP | Internet Message Access Protocol | Brute-forcing IMAP logins, often used to access email servers. | `medusa -M imap -h mail.example.com -U users.txt -P passwords.txt` |
    | MySQL | MySQL Database | Brute-forcing MySQL database credentials, commonly used for web applications and databases. | `medusa -M mysql -h 192.168.1.100 -u root -P passwords.txt` |
    | POP3 | Post Office Protocol 3 | Brute-forcing POP3 logins, typically used to retrieve emails from a mail server. | `medusa -M pop3 -h mail.example.com -U users.txt -P passwords.txt` |
    | RDP | Remote Desktop Protocol | Brute-forcing RDP logins, commonly used for remote desktop access to Windows systems. | `medusa -M rdp -h 192.168.1.100 -u admin -P passwords.txt` |
    | SSHv2 | Secure Shell (SSH) | Brute-forcing SSH logins, commonly used for secure remote access. | `medusa -M ssh -h 192.168.1.100 -u root -P passwords.txt` |
    | Subversion (SVN) | Version Control System | Brute-forcing Subversion (SVN) repositories for version control. | `medusa -M svn -h 192.168.1.100 -u admin -P passwords.txt` |
    | Telnet | Telnet Protocol | Brute-forcing Telnet services for remote command execution on older systems. | `medusa -M telnet -h 192.168.1.100 -u admin -P passwords.txt` |
    | VNC | Virtual Network Computing | Brute-forcing VNC login credentials for remote desktop access. | `medusa -M vnc -h 192.168.1.100 -P passwords.txt` |
    | Web Form | Brute-forcing Web Login Forms | Brute-forcing login forms on websites using HTTP POST requests. | `medusa -M web-form -h www.example.com -U users.txt -P passwords.txt -m FORM:"username=^USER^&password=^PASS^:F=Invalid"` |
