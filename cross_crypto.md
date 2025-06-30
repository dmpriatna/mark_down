        public void Cryptography()
        {
            /* js code
                (async () => {

                const utf8Encoder = new TextEncoder('utf-8');
                const salt = crypto.getRandomValues(new Uint8Array(16)); // Fix 1: consider salt
                const iv = crypto.getRandomValues(new Uint8Array(12));
                const iterations = 25000; // Fix 2: apply the same iteration count

                async function deriveKey(password) {
                    const buffer = utf8Encoder.encode(password);
                    const key = await crypto.subtle.importKey(
                        'raw',
                        buffer,
                        { name: 'PBKDF2' },
                        false,
                        ['deriveKey'],
                    );

                    const privateKey = crypto.subtle.deriveKey(
                        {
                            name: 'PBKDF2',
                            hash: { name: 'SHA-256' },
                            iterations,
                            salt,
                        },
                        key,
                        {
                            name: 'AES-GCM',
                            length: 256, // Fix 3: use the same key size
                        },
                        false,
                        ['encrypt', 'decrypt'],
                    );

                    return privateKey;
                }
                
                const buff_to_base64 = (buff) => btoa(String.fromCharCode.apply(null, buff));
                const base64_to_buf = (b64) => Uint8Array.from(atob(b64), (c) => c.charCodeAt(null));

                async function encrypt(key, data, iv, salt) {
                    const buffer = new TextEncoder().encode(data);

                    const privatekey = await deriveKey(key);
                    const encrypted = await crypto.subtle.encrypt(
                        {
                            name: 'AES-GCM',
                            iv,
                            tagLength: 128,
                        },
                        privatekey,
                        buffer,
                    );

                    const bytes = new Uint8Array(encrypted);
                    let buff = new Uint8Array(salt.byteLength + iv.byteLength + encrypted.byteLength);
                    buff.set(salt, 0); // Fix 1: consider salt
                    buff.set(iv, salt.byteLength);
                    buff.set(bytes, salt.byteLength + iv.byteLength);

                    const base64Buff = buff_to_base64(buff);
                    return base64Buff;
                }

                async function decrypt(key, data) {
                    const d = base64_to_buf(data);
                    const salt = d.slice(0, 16); // Fix 1: consider salt
                    const iv = d.slice(16, 16 + 12)
                    const ec = d.slice(16 + 12);

                    const decrypted = await window.crypto.subtle.decrypt(
                        {
                            name: 'AES-GCM',
                            iv,
                            tagLength: 128,
                        },
                        await deriveKey(key),
                        ec
                    );

                    return new TextDecoder().decode(new Uint8Array(decrypted));
                }

                var data = 'The quick brown fox jumps over the lazy dog';
                var passphrase = 'my passphrase';
                var ct = await encrypt(passphrase, data, iv, salt);
                var dt = await decrypt(passphrase, ct);
                console.log(ct);
                console.log(dt);

            })();
            */

            string ciphertext = "";
            Span<byte> encryptedData = Convert.FromBase64String(ciphertext).AsSpan();
            // Fix 1: consider salt (and apply the correct parameters)
            Span<byte> salt = encryptedData[..16];
            Span<byte> nonce = encryptedData[16..(16 + 12)];
            Span<byte> data = encryptedData[(16 + 12)..^16];
            Span<byte> tag = encryptedData[^16..];

            string password = "my passphrase";
            // Fix 2: apply the same iteration count
            using System.Security.Cryptography.Rfc2898DeriveBytes pbkdf2 = new(System.Text.Encoding.UTF8.GetBytes(password), salt.ToArray(), 25000, System.Security.Cryptography.HashAlgorithmName.SHA256);
            // Fix 3: use the same key size (e.g. 32 bytes for AES-256)
            using System.Security.Cryptography.AesGcm aes = new(pbkdf2.GetBytes(32), pbkdf2.GetBytes(32).Length);
            Span<byte> result = new byte[data.Length];
            aes.Decrypt(nonce, data, tag, result);

            var plainText = System.Text.Encoding.UTF8.GetString(result);
        }

        public void DecryptAesGCM(byte[] ciphertext, byte[] iv, byte[] salt, string password)
        {
            /* js code
            async function encryptMessage(message, password) {
            const enc = new TextEncoder();
            const salt = crypto.getRandomValues(new Uint8Array(16));
            const iv = crypto.getRandomValues(new Uint8Array(12));

            const keyMaterial = await crypto.subtle.importKey(
                'raw',
                enc.encode(password),
                { name: 'PBKDF2' },
                false,
                ['deriveKey']
            );

            const key = await crypto.subtle.deriveKey(
                {
                name: 'PBKDF2',
                salt,
                iterations: 100000,
                hash: 'SHA-256'
                },
                keyMaterial,
                { name: 'AES-GCM', length: 256 },
                true,
                ['encrypt']
            );

            const ciphertext = await crypto.subtle.encrypt(
                { name: 'AES-GCM', iv },
                key,
                enc.encode(message)
            );

            return {
                ciphertext: Array.from(new Uint8Array(ciphertext)),
                iv: Array.from(iv),
                salt: Array.from(salt)
            };
            }

            */

            //another version
            using var deriveBytes = new System.Security.Cryptography.Rfc2898DeriveBytes(password, salt, 100000, System.Security.Cryptography.HashAlgorithmName.SHA256);
            byte[] key = deriveBytes.GetBytes(32);

            byte[] tag = new byte[16];
            byte[] actualCipher = new byte[ciphertext.Length - tag.Length];
            Array.Copy(ciphertext, 0, actualCipher, 0, actualCipher.Length);
            Array.Copy(ciphertext, actualCipher.Length, tag, 0, tag.Length);

            byte[] plaintext = new byte[actualCipher.Length];
            using var aes = new System.Security.Cryptography.AesGcm(key, key.Length);
            aes.Decrypt(iv, actualCipher, tag, plaintext);
        }
