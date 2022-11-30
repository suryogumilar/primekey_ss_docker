# Add workers

## Crypto worker (internal)

1. Pada laman SignServer Administration Web tambahkan worker. Load dari template dengan opsi `keystore-crypto.properties`
2. Update the following in the configuration:
   * Change "WORKERGENID1.KEYSTORETYPE=PKCS12" to "WORKERGENID1.KEYSTORETYPE=INTERNAL". (Opsi INTERNAL = to use an in-configuration keystore in other words INTERNAL to use a keystore stored in the database (tied to the crypto worker))
   * Remove the line starting with "WORKERGENID1.KEYSTOREPATH".
3. set WORKERGENID1.KEYSTOREPASSWORD=[isi dengan password] cara ini membuat token "auto-activated"
4. Apply untuk menambakan Crypto token
5. Aktifasi Crypto token yang telah dibuat (atau bisa dilakukan setelah gen key. Jika tanpa melakukan aktifasi, maka langsung jalankan ke point 7)
6. Masukkan passwrd baru, jangan sampai lupa karena dibutuhkan saat aktifasi ulang saat signserver direstart atau tetapkan pada property WORKERGENID1.KEYSTOREPASSWORD=[isi dengan password] cara ini membuat token "auto-activated"
7. setelahnya masuk ke entry crypto token tersebut dan pilih tab **'Crypto Token'** untuk generate key
8. click Generate key.
9. specify a New Key Alias name for the key, for example "mrtdsod_test"
10. Click Generate dan Aktifasi Crypto token yang telah dibuat (jika point 5 terlewati) namun biasanya jika sudah genkey maka token crypto akan langsung teraktifasi.
11. and verify that the worker is now in state **ACTIVE**.


## PDF signer worker

1. Add worker 
2. choose from template
3. chooose `pdfsigner.properties`
4. fill `WORKERGENID1.NAME=[name for pdfsigner worker]`
5. fill `WORKERGENID1.CRYPTOTOKEN=[name for crypto token, wether hsm crypto token or besides that]`
6. comment the entry `WORKERGENID1.DEFAULTKEY`
7. fill `WORKERGENID1.LOCATION` entry
8. Apply
9. click the newly created pdfsigner worker, you will see it is not active with message `Error: No Signer Certificate have been uploaded to this signer.`
10. Click Renew Key
11. Select `key Algorithm` and `key specification` (example RSA 3072)
12. Fill the `New Key alias` (example: pdfsigner_sg1_key_alias_0001)
13. Click `Generate`
14. Again click the newly created pdfsigner worker, the message is still the same as before  `Error: No Signer Certificate have been uploaded to this signer.`
15. Now click generate csr
16. Key column would be filled with our filled new key alias that we have filled on the step before (example: Next key (pdfsigner_sg1_key_alias_0001))
17. fill the  DN (example:CN=pdfsigner_sg1_DN)
18. Choose `Standard CSR` for the format
19. Click `Generate`
20. Download the CSR
21. the file downloaded would be something like this `PDFSigner_sg1-pdfsigner_sg1_key_alias_0001.p10`
22. send to CA or RA to be signed
24. Select the signer in the SignServer Workers list, and click `Install Certificates`.
25. Browse for the PEM certificate file and click `Add`.
26. Click `Install` and confirm that the signer is now listed as ACTIVE and ready to be used.
27 test by signing a pdf files and check its certificate