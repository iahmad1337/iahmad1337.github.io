# TLPI Page

## Meeting
Scheduled on 11:00 (Moscow, GMT+3) every sunday
<form id="xorForm">
        <label for="inputText">Enter password:</label><br>
        <input type="text" id="inputText" name="inputText" size="50"><br><br>
        <button type="submit">Convert</button>
    </form>

<div id="output"></div>

<script>
    // Fixed byte array for XOR operation (you can change this)
    const encodedResult = [4, 29, 26, 5, 0, 23, 77, 78, 26, 7, 93, 78, 7, 9, 67, 10, 15, 25, 31, 2, 8, 14, 5, 2, 92, 0, 41, 33, 8, 14, 3, 34, 22, 111, 82, 52, 43, 94, 61, 26, 89, 80, 6, 80, 89, 4, 11, 69, 3, 7, 34, 5, 10, 122, 94, 86, 88, 29, 6, 2, 35, 120, 80, 81, 1, 3, 23, 100];

    document.getElementById('xorForm').addEventListener('submit', function(e) {
        e.preventDefault();
        
        // Get input text
        const inputText = document.getElementById('inputText').value.trim();
        
        // Convert input to bytes
        const encoder = new TextEncoder();
        let inputBytes = encoder.encode(inputText);
        if (inputBytes.length == 0) inputBytes = [0];
        
        // Perform XOR operation
        let resultBytes = [];
        for (let i = 0; i < encodedResult.length; i++) {
            resultBytes.push(inputBytes[i % inputBytes.length] ^ encodedResult[i]);
        }
        
        // const crypto = require('crypto');
        window.crypto.subtle.digest("SHA-256", new Uint8Array(resultBytes)).then(hashBuffer => {
            // const hashBuffer = window.crypto.subtle.digest("SHA-256", new Uint8Array(resultBytes));
            const hashArray = Array.from(new Uint8Array(hashBuffer));
            const hash = hashArray
                .map((item) => item.toString(16).padStart(2, "0"))
                .join("");


            console.log(hashBuffer)
            console.log(hash)

            const outputDiv = document.getElementById('output');
            if (hash !== '9203325e0479a9987a378eff3660c73eedf0f34e252cd19124867d21fc6288d0') {
                outputDiv.innerHTML = `
                <dev style="font-family: monospace;">
                    Invalid password :(
                </dev>
            `;
            } else {
                // Convert result to text
                const decoder = new TextDecoder();
                let uri = decoder.decode(Uint8Array.from(resultBytes));
            
                outputDiv.innerHTML = `
                    <h3>Your link:</h3>
                    <a href="${uri}" onclick="return false;" style="font-family: monospace;">
                        ${uri}
                    </a>
                `;
            }
        });
    });
</script>



## Chapter 56 "Sockets: Introduction"
Notes from 07.09.2025

- One of the possible problems of using sockets POSIX API are [strict aliasing rules](https://stackoverflow.com/questions/98650/what-is-the-strict-aliasing-rule) that are very hard to follow
- Example of how the solutions to exercises can be organized: https://github.com/iahmad1337/linux-learning/tree/main/linux-programming-interface/chapters

How to get LSP support for the code in the book:
```
    8  wget https://man7.org/tlpi/code/download/tlpi-250328-dist.tar.gz
    9  ls
   10  tar -xf tlpi-250328-dist.tar.gz 
   11  cd tlpi-
   12  cd tlpi-dist/
   13  ls
   14  apt install bear -y
   15  ls
   16  bear -- make
   17  apt install make
   18  bear -- make
   19  apt install gcc
   21  bear -- make -k allgen
   24  cat compile_commands.json
```

## Chapter 57 "Sockets: UNIX Domain"

Meeting is planned on 28.09.2025 at 11:00 (Moscow time)
