# Add workers


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