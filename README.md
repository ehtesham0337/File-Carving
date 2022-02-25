# File Carving - Forensics
We are given a pdf file called `Mandiant.pdf` that contains an EmbeddedFile object stream. The pdf is 76 pages long and we could read it but lets move on to its analysis.
# PDF Analysis
First I used `qpdf` to extract information about the pdf.
> ` qpdf --qdf --object-streams=disable Mandiant_c920fc463eaf996489749457abc9b2eb.pdf out.pdf`

Now if we open `out.pdf` in any text editor of choice, we can look up `Embeddedfile` for a hint. 
Aha! A stream of base64 text. Delete everything above and below this text and save the file as `out.b64`.

Now decode and output it to a file `out`. 
> `base64 -d < out.b64 > out`

Using the command 
> `file out` , we see that its a PNG.

> `mv out out.png`



Run scalpel on this PNG.
> `scalpel -c /etc/scalpel/scalpel.conf -o scaltest out.png`

The PNG contains a 7zip archive. Move into `7z-10-0` and we can extract `00000000.7z` using `p7zip`. 
> `7za x 00000000.7z `

We get a text file called `secret.exe` which seems to be a bunch of base64. 

Decoding it,
> `base64 -d < secret.exe > pic.jpg`

Running `file pic.jpg` tells us its a JPEG.
> pic: JPEG image data


Using `strings`, we find that the JPG contains two base64 encoded blobs:
> `strings -a pic.jpg`
 
> b#o]
  Y2EB
  V~D
  [...]
  "aKxw
  S#UH
  VVGEC8uGDQuNhhk6FKg0ICF9jVAUS54zurveSzXcwE9MsIHIZPuvP6vrSDgwULy5Kvm/wPe3zxddM4SSPgvWIg==
  XQN6Y6QIpfofWw857i5DK[...]  


We can decode and split these into two and run `xxd` but we dont find anything.

We have to use a program called `Free File Camouflage` to extract a hidden file, `a.out`, from the blob(s) - an ELF. Lets run it,
> `./a.out`

But it denies us permission even if sudo is used. Using `chmod`,

> `chmod +x a.out`

By running the file again, we obtain the flag.

> `hello world, i found this flag under some bit-maps.... [HAVE A FLAG]
  
  
> flag{s3v3r4l_l4y3r5_d33p_&_2m4ny_l4yers_w1d3}
`


