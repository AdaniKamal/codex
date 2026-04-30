```zsh

┌──(kali㉿adanikamal)-[~/Desktop/test]
└─$ codex --yolo "breakthrough and perform steganography on the Lahad_Datu.docx file and find the flag" 
╭─────────────────────────────────────────────────╮
│ ✨ Update available! 0.113.0 -> 0.125.0         │
│ Run npm install -g @openai/codex to update.     │
│                                                 │
│ See full release notes:                         │
│ https://github.com/openai/codex/releases/latest │
╰─────────────────────────────────────────────────╯

╭───────────────────────────────────────────────────╮
│ >_ OpenAI Codex (v0.113.0)                        │
│                                                   │
│ model:     gpt-5.3-codex xhigh   /model to change │
│ directory: ~/Desktop/test                         │
╰───────────────────────────────────────────────────╯

  Tip: Update Required - This version will no longer be supported starting May 8th. Please upgrade to the latest version
  (https://github.com/openai/codex/releases/latest) using your preferred package manager.


› breakthrough and perform steganography on the Lahad_Datu.docx file and find the flag


• I’ll inspect Lahad_Datu.docx and its embedded contents first, then run common steganography checks on any extracted media/metadata to locate the flag.
  I’m starting by listing files and probing the DOCX structure.

• Explored
  └ List ls -la

• Ran file Lahad_Datu.docx && exiftool Lahad_Datu.docx || true
  └ ExifTool Version Number         : 13.25
    File Name                       : Lahad_Datu.docx
    … +7 lines
    File Type Extension             : docx
    MIME Type                       : application/vnd.openxmlformats-officedocument.wordprocessingml.document

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• The DOCX is encrypted (CDFV2 Encrypted), so the next step is password recovery. I’ll extract its hash and try quick cracking paths (wordlist/rules)
  before deeper brute force.

• Ran which office2john.py office2john john hashcat fcrackzip 7z libreoffice soffice || true
  └ office2john.py not found
    /usr/bin/office2john
    … +4 lines
    libreoffice not found
    soffice not found

• Ran strings -n 6 Lahad_Datu.docx | head -n 80
  └ <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
    <encryption xmlns="http://schemas.microsoft.com/office/2006/encryption" xmlns:p="http://schemas.microsoft.com/office/2006/keyEncryptor/password"
    … +78 lines
    ;t2!&k>
    %18GZo=k

• Ran office2john Lahad_Datu.docx > lahad.hash && cat lahad.hash
  └ Lahad_Datu.docx:
    $office$*2013*100000*256*16*1341e4f573865438d99a5fba68ba4b3c*f3481ea45a0479c4250b42dcdfd74fc3*72a2d75abb1d158780c588607affc6b294a1fa25215855981399d3
    628ec18997

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• I extracted the Office 2013 hash successfully. I’ll run targeted wordlist attacks first (common passwords and likely Malaysia/Lahad-Datu terms), then
  escalate to broader cracking if needed.

• Ran ls /usr/share/wordlists 2>/dev/null | head
  └ amass
    dirb
    … +6 lines
    metasploit
    nmap.lst

• Ran john --wordlist=/usr/share/wordlists/rockyou.txt lahad.hash --format=office --fork=4
  └ Using default input encoding: UTF-8
    Loaded 1 password hash (Office, 2007/2010/2013 [SHA1 512/512 AVX512BW 16x / SHA512 512/512 AVX512BW 8x AES])
    No password hashes left to crack (see FAQ)

• Ran cat > custom_words.txt << 'EOF'
  │ lahaddatu
  │ lahad_datu
  │ … +24 lines
  └ Using default input encoding: UTF-8
    Loaded 1 password hash (Office, 2007/2010/2013 [SHA1 512/512 AVX512BW 16x / SHA512 512/512 AVX512BW 8x AES])
    No password hashes left to crack (see FAQ)

• Ran john --show --format=office lahad.hash
  └ Lahad_Datu.docx:playboy

    1 password hash cracked, 0 left

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• I recovered the DOCX password from the local John cache: playboy. Next I’ll decrypt/extract the document contents and inspect embedded media/objects
  for hidden data and the flag.

• Ran which msoffcrypto-tool python3 olevba olevba.py binwalk steghide zsteg exiftool foremost || true
  └ /usr/local/bin/msoffcrypto-tool
    /usr/bin/python3
    … +5 lines
    /usr/bin/exiftool
    /usr/bin/foremost

• Ran rm -rf lahad_work && mkdir lahad_work && cd lahad_work && 7z x -pplayboy ../Lahad_Datu.docx >/tmp/7z_lahad.log && ls -la && echo '---7z log---' &&
  │ cat /tmp/7z_lahad.log
  └ total 88
    drwxrwxr-x 3 kali kali  4096 Apr 30 04:47 .
    … +32 lines
    Size:       73941
    Compressed: 79360

• Ran rm -rf decrypted_unzip && mkdir decrypted_unzip && cd decrypted_unzip && 7z x ../decrypted.docx >/tmp/decrypted_7z.log && find . -maxdepth 4 -type
  │ f | sort && echo '---' && cat /tmp/decrypted_7z.log
  └
    ERROR: errno=2 : No such file or directory
    … +4 lines
    System ERROR:
    errno=2 : No such file or directory

• Ran msoffcrypto-tool -p playboy Lahad_Datu.docx decrypted.docx && file decrypted.docx && ls -lh decrypted.docx
  └ Traceback (most recent call last):
      File "/usr/local/bin/msoffcrypto-tool", line 5, in <module>
        from msoffcrypto.__main__ import main
    ModuleNotFoundError: No module named 'msoffcrypto'

• The installed msoffcrypto-tool wrapper is broken (missing Python module), so I’m fixing that first by installing the module locally, then I’ll decrypt
  the file and continue steg analysis.

• Ran python3 -m pip install --user --quiet msoffcrypto-tool && python3 -c "import msoffcrypto,sys; print(msoffcrypto.__version__)"
  └ error: externally-managed-environment

    … +19 lines
    breaking your Python installation or OS, by passing --break-system-packages.
    hint: See PEP 668 for the detailed specification.

• Ran python3 -m venv .venv_mso && .venv_mso/bin/pip install --quiet msoffcrypto-tool && .venv_mso/bin/python -c "import msoffcrypto; print('ok')"
  └ ok

• Ran rm -rf decrypted_unzip && mkdir decrypted_unzip && cd decrypted_unzip && 7z x ../decrypted.docx >/tmp/decrypted_7z.log && find . -maxdepth 5 -type
  │ f | sort
  └
    ERROR: errno=2 : No such file or directory
    … +4 lines

• Ran .venv_mso/bin/msoffcrypto-tool -p playboy Lahad_Datu.docx decrypted.docx && file decrypted.docx && ls -lh decrypted.docx
  └ decrypted.docx: Microsoft Word 2007+
    -rw-rw-r-- 1 kali kali 71K Apr 30 04:48 decrypted.docx

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• Decryption worked (decrypted.docx created); the unzip step failed only due to parallel timing, so I’m re-running extraction and then checking embedded
  files for hidden payloads.

• Ran rm -rf decrypted_unzip && mkdir decrypted_unzip && cd decrypted_unzip && 7z x ../decrypted.docx >/tmp/decrypted_7z.log && find . -maxdepth 6 -type
  │ f | sort && echo '---7z---' && sed -n '1,80p' /tmp/decrypted_7z.log
  └ ./[Content_Types].xml
    ./_rels/.rels
    … +37 lines
    Size:       137504
    Compressed: 72185

• Explored
  └ Search flag|ctf|lahad|datu|playboy|password|key|{.*} in decrypted_unzip

• Ran exiftool decrypted_unzip/word/media/image1.jpeg
  └ ExifTool Version Number         : 13.25
    File Name                       : image1.jpeg
    … +15 lines
    Image Size                      : 600x450
    Megapixels                      : 0.270

• Ran file decrypted_unzip/word/media/image1.jpeg && binwalk decrypted_unzip/word/media/image1.jpeg
  └
    DECIMAL       HEXADECIMAL     DESCRIPTION
    --------------------------------------------------------------------------------


────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• I’ve confirmed there’s one embedded JPEG (image1.jpeg) with no obvious EXIF/binwalk clues yet. I’m now checking the DOCX XML content for hidden
  strings/hints (including possible steghide passphrase) and then I’ll run image steg extraction.

• Explored
  └ Search flag|ctf|pass|password|key|hint|lahad|datu|steg|hidden in decrypted_unzip
    Read document.xml, custom.xml

• Ran steghide info decrypted_unzip/word/media/image1.jpeg << 'EOF'
  │
  │ EOF
  └ Try to get information about embedded data ? (y/n) steghide: could not get terminal attributes.
    "image1.jpeg":
      format: jpeg
      capacity: 2.8 KB

• Ran steghide info -p "" decrypted_unzip/word/media/image1.jpeg
  └ "image1.jpeg":
      format: jpeg
      capacity: 2.8 KB
    steghide: could not extract any data with that passphrase!

• Ran steghide extract -sf decrypted_unzip/word/media/image1.jpeg -p "" -f -xf steg_out_empty 2>&1 | sed -n '1,120p'
  └ steghide: could not extract any data with that passphrase!

• Ran for p in playboy JamalulKiramIII jamalulkiramiii LahadDatu lahaddatu 3108 2013 Sabah; do echo "== $p =="; steghide extract -sf decrypted_unzip/
  │ word/media/image1.jpeg -p "$p" -f -xf "steg_$p.out" 2>&1 | head -n 2; done
  └ steghide: could not extract any data with that passphrase!
    == Sabah ==
    steghide: could not extract any data with that passphrase!

• Ran which stegseek stegcracker outguess exiftool binwalk foremost strings convert identify jpeginfo jsteg || true
  └ /usr/bin/stegseek
    stegcracker not found
    … +7 lines
    jpeginfo not found
    /usr/bin/jsteg

─ Worked for 1m 14s ────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• I found a suspicious flag-like token directly in word/document.xml, but I’m validating whether an additional hidden payload exists in the JPEG before
  finalizing. I’ll run stegseek/jsteg plus carving checks to confirm.

• Ran strings -n 4 decrypted_unzip/word/media/image1.jpeg | tail -n 80
  └ YsrjU
    M/D@
    … +76 lines
    G"]J(I
    %+Tq

• Ran xxd -g 1 decrypted_unzip/word/media/image1.jpeg | tail -n 40
  └ 0000c280: 4f b6 cf d5 68 eb 44 9c 85 36 39 32 f8 f5 47 28  O...h.D..692..G(
    0000c290: c8 95 7b 6d 48 48 44 3a 2f fa 59 3f 4e cd 96 59  ..{mHHD:/.Y?N..Y
    … +36 lines
    0000c4e0: 3e fb be c4 7f 50 fb 0b 74 2d d8 88 fa 97 a1 8b  >....P..t-......
    0000c4f0: 78 f7 1e df ff d9                                x.....

• Ran rm -rf carve_out && mkdir carve_out && foremost -i decrypted_unzip/word/media/image1.jpeg -o carve_out >/tmp/foremost.log 2>&1; echo 'foremost
  │ done'; find carve_out -type f | sort; echo '---'; sed -n '1,120p' /tmp/foremost.log
  └ foremost done
    carve_out/audit.txt
    … +2 lines
    Processing: decrypted_unzip/word/media/image1.jpeg
    |*|

• Ran which stegoveritas stegolsb-cli stegolsb stegoveritas.py || true
  └ /home/kali/.local/bin/stegoveritas
    stegolsb-cli not found
    stegolsb not found
    stegoveritas.py not found

• Ran jsteg 2>&1 | head -n 50
  └ Usage:
        jsteg hide in.jpg [FILE] [out.jpg]
          Hide FILE (or stdin) in in.jpg, writing the result to out.jpg (or stdout)
        jsteg reveal in.jpg [FILE]
          Write the hidden contents of in.jpg to FILE (or stdout)

• Ran zsteg -a decrypted_unzip/word/media/image1.jpeg 2>&1 | head -n 80
  └ [!] #<ZPNG::NotSupported: Unsupported header "\xFF\xD8\xFF\xE1\x00\x02\xFF\xE1" in #<File:decrypted_unzip/word/media/image1.jpeg>>

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• I’m now testing JPEG-specific steg tools (jsteg, stegseek, stegoveritas) to confirm whether the image contains a second-stage hidden payload or if the
  XML token is the final flag.

• Ran jsteg reveal decrypted_unzip/word/media/image1.jpeg > jsteg_reveal.bin 2>/tmp/jsteg.err; echo "exit:$?"; ls -lh jsteg_reveal.bin 2>/dev/null ||
  │ true; sed -n '1,80p' /tmp/jsteg.err; file jsteg_reveal.bin 2>/dev/null || true; strings -n 4 jsteg_reveal.bin 2>/dev/null | head -n 40
  └ exit:1
    -rw-rw-r-- 1 kali kali 0 Apr 30 04:51 jsteg_reveal.bin
    could not decode jpeg:unsupported JPEG feature: progressive decoding
    jsteg_reveal.bin: empty

• Ran stegoveritas decrypted_unzip/word/media/image1.jpeg -o stegoveritas_out >/tmp/stegoveritas.log 2>&1; echo "exit:$?"; find stegoveritas_out
  │ -maxdepth 4 -type f | sort | sed -n '1,120p'; echo '---log---'; sed -n '1,160p' /tmp/stegoveritas.log
  └ exit:1
    find: ‘stegoveritas_out’: No such file or directory
    … +3 lines
        from stegoveritas.stegoveritas import main
    ModuleNotFoundError: No module named 'stegoveritas'

• Ran stegseek decrypted_unzip/word/media/image1.jpeg doc_words.txt -xf stegseek_docwords.out > /tmp/stegseek_docwords.log 2>&1; echo "exit:$?"; sed -n
  │ '1,160p' /tmp/stegseek_docwords.log; ls -lh stegseek_docwords.out 2>/dev/null || true; file stegseek_docwords.out 2>/dev/null || true; strings -n
  │ 4 stegseek_docwords.out 2>/dev/null | head -n 40
  └ exit:1


• Ran python3 - << 'PY'
  │ import re
  │ text=open('decrypted_unzip/word/document.xml','r',encoding='utf-8').read()
  │ … +12 lines
  └ 286
    286 doc_words.txt
    … +38 lines
    2017
    model3d

• Ran if [ -f /usr/share/wordlists/rockyou.txt ]; then echo ROCKYOU_EXISTS; wc -l /usr/share/wordlists/rockyou.txt; elif [ -f /usr/share/wordlists/
  │ rockyou.txt.gz ]; then echo ROCKYOU_GZ_EXISTS; ls -lh /usr/share/wordlists/rockyou.txt.gz; else echo ROCKYOU_MISSING; fi
  └ 14344392 /usr/share/wordlists/rockyou.txt

• Ran pwd; ls -l doc_words.txt; stegseek decrypted_unzip/word/media/image1.jpeg ./doc_words.txt -xf stegseek_docwords.out > /tmp/stegseek_docwords.log
  │ 2>&1; echo "exit:$?"; sed -n '1,120p' /tmp/stegseek_docwords.log
  └ /home/kali/Desktop/test
    … +3 lines
    [!] error: Could not find a valid passphrase.

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• The image does appear to be steghide-compatible, but no passphrase was found in document-derived words. I’m launching a broader rockyou crack now to
  verify whether there’s any hidden payload at all.

• Ran time stegseek decrypted_unzip/word/media/image1.jpeg /usr/share/wordlists/rockyou.txt -xf stegseek_rockyou.out > /tmp/stegseek_rockyou.log 2>&1;
  │ || true; strings -n 4 stegseek_rockyou.out 2>/dev/null | head -n 80
  └ stegseek decrypted_unzip/word/media/image1.jpeg  -xf stegseek_rockyou.out >    7.55s user 0.19s system 347% cpu 2.229 total
    exit:1
    … +18 lines
    [!] error: Could not find a valid passphrase.
    stegseek_rockyou.out: cannot open `stegseek_rockyou.out' (No such file or directory)

• Searched "3108{0Y3R4E1_D4FF4E}"

• Explored
  └ Search vanish|specVanish|w:color|w:sz|w:highlight|w:rPr|w:lang|w:rtl|w:spacing in document.xml

• Ran xmllint --format decrypted_unzip/word/document.xml > /tmp/doc_fmt.xml && sed -n '120,240p' /tmp/doc_fmt.xml
  └       </w:r>
          <w:r w:rsidR="00044DE1" w:rsidRPr="00044DE1">
    … +13 lines
      </w:body>
    </w:document>

• Ran python3 - << 'PY'
  │ import xml.etree.ElementTree as ET
  │ ns={'w':'http://schemas.openxmlformats.org/wordprocessingml/2006/main'}
  │ … +13 lines
  └ 002 | SHAH ALAM - Peristiwa Lahad Datu meninggalkan sejarah hitam buat Malaysia apabila ramai perwira negara gugur mempertahankan kedaulatan negara
    daripada dicerobohi pengganas Sulu pada 2013. | -
    … +10 lines
    012 | } | -
    013 | . | -

• Searched "D4FF4E"

• Searched "3108" "Lahad Datu" "flag"

• Ran python3 - << 'PY'
  │ import string
  │ ct='0Y3R4E1_D4FF4E'
  │ … +25 lines
  └ dec 0P3R4S1_D4UL4T
    enc 0H3R4Q1_D4QZ4P

• Searched "0P3R4S1_D4UL4T"

• Ran python3 - << 'PY'
  │ import string
  │ ct='0Y3R4E1_D4FF4E'
  │ … +12 lines
  └ 3108{0P3R4S1_D4UL4T}

─ Worked for 2m 36s ────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• Flag found: 3108{0P3R4S1_D4UL4T}

  I cracked Lahad_Datu.docx (password: playboy), decrypted it, then extracted the obfuscated token from document.xml: 3108{0Y3R4E1_D4FF4E}.
  Using JamalulKiramIII (from the same doc) as the Vigenere key decodes it to the final flag above.
 
 
› Use /skills to list available skills
 
  gpt-5.3-codex xhigh · 84% left · 

```
