---
title: ""
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

まずは、暗号化に必要な鍵と初期化ベクトル(IV)を生成します。
鍵の生成には、高度な乱数生成アルゴリズムを使用することが重要です。以下のスクリプトは、256ビット（32バイト）の鍵を生成します。

```powershell
$provider = New-Object System.Security.Cryptography.RNGCryptoServiceProvider
$key = New-Object byte[](32)
$provider.GetBytes($key)
$keybase64 = [System.Convert]::ToBase64String($key)
Write-Output $keybase64
```

次に、IVを生成します。
以下のスクリプトはIVを生成します。

```powershell
$aes = New-Object System.Security.Cryptography.AesCryptoServiceProvider
$iv = $aes.IV
$ivbase64 = [System.Convert]::ToBase64String($iv)
Write-Output $ivbase64
```

最後に、生成した鍵とIVを使って、パスワードを暗号化するスクリプトを実行します。

```powershell
# import System.Security.Cryptography assembly
Add-Type -AssemblyName System.Security.Cryptography

# define encryption key and initialization vector
$key = [System.Convert]::FromBase64String("YourEncryptionKeyGoesHere")
$iv = [System.Convert]::FromBase64String("YourInitializationVectorGoesHere")

# read password from file
$password = Get-Content "password.txt"

# create AES256 encryptor
$aes = New-Object System.Security.Cryptography.AesCryptoServiceProvider
$aes.Key = $key
$aes.IV = $iv
$aes.Padding = "PKCS7"
$aes.Mode = "CBC"

# create encryptor and stream objects
$encryptor = $aes.CreateEncryptor()
$stream = New-Object System.IO.MemoryStream

# create crypto stream
$cryptoStream = New-Object System.Security.Cryptography.CryptoStream $stream, $encryptor, "Write"

# create stream writer
$streamWriter = New-Object System.IO.StreamWriter $cryptoStream

# write password to stream
$streamWriter.Write($password)

# close stream writer
$streamWriter.Close()

# close crypto stream
$cryptoStream.Close()

# close memory stream
$stream.Close()

# convert encrypted data to base64 string
$encryptedPassword = [System.Convert]::ToBase64String($stream.
```


## もとになったやりとり
![](/images/encrypt_textfile_chatgpt.png)  