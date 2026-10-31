# LAPORAN-PHPINFO

# Penetration Testing Report - PHP Information Disclosure
**Date:** 31 Mei 2026
**Tester:** debker (Blangpidie, Aceh)
**Target:** http://localhost:8080 (Docker php:8.2-apache)
**Tool:** Nuclei v3.11.1

## 1. Executive Summary
Ditemukan kerentanan Information Disclosure pada file `info.php` yang mengekspos `phpinfo()`. 

Severity: Low (CVSS 3.7) - tapi sering jadi stepping stone ke RCE.

## 2. Vulnerability Details
- Name: PHPInfo Exposure
- Template: http/exposures/configs/phpinfo-files.yaml
- URL: http://localhost:8080/info.php
- Version Detected: PHP 8.2.35

### Proof of Concept

Step 1 - Create Vulnerable:
sudo docker run -d -p 8080:80 --name php-lab php:8.2-apache
echo "<?php phpinfo(); ?>" > /tmp/info.php
sudo docker cp /tmp/info.php php-lab:/var/www/html/info.php

Step 2 - Detection:
nuclei -u http://localhost:8080 -t ~/.local/nuclei-templates/http/exposures/configs/phpinfo-files.yaml -v

Output VULNERABLE:
[phpinfo-files] [http] [low] http://localhost:8080/info.php ["8.2.35"]
1 matches found.

Step 3 - Manual Verify:
curl -s http://localhost:8080/info.php | grep "PHP Version"

## 3. Impact
- Tau versi PHP 8.2.35 (cari CVE spesifik)
- Terlihat DOCUMENT_ROOT, SERVER_SOFTWARE
- Dipakai untuk exploit chain ke LFI/RCE

## 4. Remediation
Immediate Fix:
sudo docker exec php-lab rm /var/www/html/info.php

Verification After Fix:
curl -s http://localhost:8080/info.php -> 404 Not Found
nuclei -> No results found.

Best Practice:
1. Jangan upload info.php ke production
2. Block di .htaccess
3. Setup nuclei di CI/CD

## 5. Conclusion
Lab berhasil: Create -> Detect -> Remediate -> Verify
