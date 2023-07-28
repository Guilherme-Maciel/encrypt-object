# ENCRYPT OBJECT EXAMPLE

O exemplo a seguir mostra uma forma de criptografia simétrica (onde a mesma chave é usada no ato de criptografar e no ato de descriptografar) denominada AES em modo CBC (Clipher Block Chaining).

O AES pode trabalhar com chaves de tamanhos de 128, 192 e 256 bits

Ele opera em blocos de dados de 128 bits e cada bloco é criptografado de forma independente dos outros blocos.

Ele embaralha os dados usando uma combinação de substituições e permutações.

Cada tamanho de chave utiliza um número de rodadas diferentes.

o CBC é uma forma de operação usada em algoritmos de criptografia simétrica para encriptação de dados. Ele une os blocos não criptografados ao que foi criptografado anteriormente

## ENCRIPTAR

```csharp
private static string EncryptString(string texto, string chave)
    {
        byte[] chaveBytes = Encoding.UTF8.GetBytes(chave);
        byte[] textoBytes = Encoding.UTF8.GetBytes(texto);
				//Retorna nova instância do AES
        using (Aes aesAlg = Aes.Create())
        {
						//Define a chave sendo utilizada
            aesAlg.Key = chaveBytes;
						//Define o modo que ira ocorrer a criptografia
            aesAlg.Mode = CipherMode.CBC;
						//bytes são adicionados ao texto claro para que o tamanho total seja um 
						//múltiplo do tamanho do bloco.
            aesAlg.Padding = PaddingMode.PKCS7;

            // Gerar um IV (vetor de inicialização) aleatório
						//Bloco de dados a ser combinado com o primeiro bloco de texto
            aesAlg.GenerateIV();

            // Criar um encriptador para executar a transformação
            ICryptoTransform encryptor = aesAlg.CreateEncryptor(aesAlg.Key, aesAlg.IV);

            // Encriptar os dados
            byte[] encryptedBytes;
            using (MemoryStream msEncrypt = new MemoryStream())
            {
                using (CryptoStream csEncrypt = new CryptoStream(msEncrypt, encryptor, CryptoStreamMode.Write))
                {
                    csEncrypt.Write(textoBytes, 0, textoBytes.Length);
                    csEncrypt.FlushFinalBlock();
                    encryptedBytes = msEncrypt.ToArray();
                }
            }

            // Concatenar o IV com os dados encriptados
            byte[] resultBytes = new byte[aesAlg.IV.Length + encryptedBytes.Length];
            Buffer.BlockCopy(aesAlg.IV, 0, resultBytes, 0, aesAlg.IV.Length);
            Buffer.BlockCopy(encryptedBytes, 0, resultBytes, aesAlg.IV.Length, encryptedBytes.Length);

            // Retornar a string encriptada em formato de Base64
            return Convert.ToBase64String(resultBytes);
        }
    }
```

## DESENCRIPTAR

```csharp
private static string DecryptString(string textoEncriptado, string chave)
{
    byte[] chaveBytes = Encoding.UTF8.GetBytes(chave);
    byte[] encryptedBytes = Convert.FromBase64String(textoEncriptado);

    using (Aes aesAlg = Aes.Create())
    {
        aesAlg.Key = chaveBytes;
        aesAlg.Mode = CipherMode.CBC;
        aesAlg.Padding = PaddingMode.PKCS7;

        // Separar o IV dos dados encriptados
        byte[] iv = new byte[aesAlg.IV.Length];
        byte[] encryptedData = new byte[encryptedBytes.Length - aesAlg.IV.Length];
        Buffer.BlockCopy(encryptedBytes, 0, iv, 0, aesAlg.IV.Length);
        Buffer.BlockCopy(encryptedBytes, aesAlg.IV.Length, encryptedData, 0, encryptedData.Length);

        // Criar um desencriptador para executar a transformação
        ICryptoTransform decryptor = aesAlg.CreateDecryptor(aesAlg.Key, iv);

        // Desencriptar os dados
        byte[] decryptedBytes;
        using (MemoryStream msDecrypt = new MemoryStream(encryptedData))
        {
            using (CryptoStream csDecrypt = new CryptoStream(msDecrypt, decryptor, CryptoStreamMode.Read))
            {
                using (MemoryStream msOutput = new MemoryStream())
                {
                    csDecrypt.CopyTo(msOutput);
                    decryptedBytes = msOutput.ToArray();
                }
            }
        }

        // Retornar a string desencriptada
        return Encoding.UTF8.GetString(decryptedBytes);
    }
}
```

## MAIN

```csharp
public static void Main()
{
		//Chave guardada em cofre
    string chave = "a4gr5s6h9jd8kf3g7a2q1w0q0m5x7dfg";
    // Criar um objeto simples para demonstração
    var pessoa = new Pessoa(){ Nome = "João", Idade = 30 };
    //Converter objeto em string
    string pessoaJson = JsonSerializer.Serialize(pessoa);
    Console.WriteLine("Objeto em string JSON: " + pessoaJson);
		//Encriptar a string do objeto
    string textoEncriptado = EncryptString(pessoaJson, chave);
    Console.WriteLine("Encriptado: " + textoEncriptado);
		//Desencriptar a string do objeto
    string textoDescriptografado = DecryptString(textoEncriptado, chave);
    Console.WriteLine("Texto Descriptografado: " + textoDescriptografado);
    //Desconverter de string para objeto
    object pessoaObj = JsonSerializer.Deserialize<Pessoa>(textoDescriptografado);
}
```