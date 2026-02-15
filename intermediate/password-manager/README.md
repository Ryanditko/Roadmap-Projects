# Password Manager

**Level:** Intermediate | **Time:** 1-2 weeks

## Description
Secure password manager with encryption, password generation, strength analysis, and vault management. Learn cryptography, secure storage, and password best practices.

## Core Features
- Master password authentication
- AES-256 encryption
- Password generator
- Strength meter
- Categories/tags
- Search and filter
- Auto-fill (browser extension)
- Encrypted export/import

## Implementation Guide

```javascript
class PasswordManager {
    constructor() {
        this.vault = [];
        this.masterPassword = null;
        this.isUnlocked = false;
    }
    
    async setMasterPassword(password) {
        const salt = crypto.getRandomValues(new Uint8Array(16));
        const key = await this.deriveKey(password, salt);
        
        localStorage.setItem('salt', this.arrayToBase64(salt));
        localStorage.setItem('masterHash', await this.hashPassword(password));
        
        this.masterPassword = key;
        this.isUnlocked = true;
    }
    
    async unlock(password) {
        const storedHash = localStorage.getItem('masterHash');
        const hash = await this.hashPassword(password);
        
        if (hash === storedHash) {
            const salt = this.base64ToArray(localStorage.getItem('salt'));
            this.masterPassword = await this.deriveKey(password, salt);
            this.isUnlocked = true;
            await this.loadVault();
            return true;
        }
        return false;
    }
    
    async deriveKey(password, salt) {
        const encoder = new TextEncoder();
        const keyMaterial = await crypto.subtle.importKey(
            'raw',
            encoder.encode(password),
            'PBKDF2',
            false,
            ['deriveBits', 'deriveKey']
        );
        
        return await crypto.subtle.deriveKey(
            {
                name: 'PBKDF2',
                salt: salt,
                iterations: 100000,
                hash: 'SHA-256'
            },
            keyMaterial,
            { name: 'AES-GCM', length: 256 },
            false,
            ['encrypt', 'decrypt']
        );
    }
    
    async encrypt(text) {
        const encoder = new TextEncoder();
        const data = encoder.encode(text);
        const iv = crypto.getRandomValues(new Uint8Array(12));
        
        const encrypted = await crypto.subtle.encrypt(
            { name: 'AES-GCM', iv: iv },
            this.masterPassword,
            data
        );
        
        return {
            iv: this.arrayToBase64(iv),
            data: this.arrayToBase64(new Uint8Array(encrypted))
        };
    }
    
    async decrypt(encrypted) {
        const iv = this.base64ToArray(encrypted.iv);
        const data = this.base64ToArray(encrypted.data);
        
        const decrypted = await crypto.subtle.decrypt(
            { name: 'AES-GCM', iv: iv },
            this.masterPassword,
            data
        );
        
        const decoder = new TextDecoder();
        return decoder.decode(decrypted);
    }
    
    generatePassword(length = 16, options = {}) {
        const lowercase = 'abcdefghijklmnopqrstuvwxyz';
        const uppercase = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
        const numbers = '0123456789';
        const symbols = '!@#$%^&*()_+-=[]{}|;:,.<>?';
        
        let chars = lowercase;
        if (options.uppercase !== false) chars += uppercase;
        if (options.numbers !== false) chars += numbers;
        if (options.symbols !== false) chars += symbols;
        
        const array = new Uint32Array(length);
        crypto.getRandomValues(array);
        
        let password = '';
        for (let i = 0; i < length; i++) {
            password += chars[array[i] % chars.length];
        }
        
        return password;
    }
    
    checkPasswordStrength(password) {
        let score = 0;
        
        if (password.length >= 8) score++;
        if (password.length >= 12) score++;
        if (password.length >= 16) score++;
        if (/[a-z]/.test(password) && /[A-Z]/.test(password)) score++;
        if (/\d/.test(password)) score++;
        if (/[^a-zA-Z0-9]/.test(password)) score++;
        
        const strengths = ['Very Weak', 'Weak', 'Fair', 'Good', 'Strong', 'Very Strong'];
        return {
            score,
            strength: strengths[score],
            percentage: (score / 6) * 100
        };
    }
    
    async addPassword(site, username, password) {
        const encrypted = await this.encrypt(password);
        
        const entry = {
            id: Date.now().toString(),
            site,
            username,
            password: encrypted,
            createdAt: new Date().toISOString()
        };
        
        this.vault.push(entry);
        await this.saveVault();
    }
    
    async saveVault() {
        const vaultData = JSON.stringify(this.vault);
        const encrypted = await this.encrypt(vaultData);
        localStorage.setItem('vault', JSON.stringify(encrypted));
    }
    
    async loadVault() {
        const encryptedVault = localStorage.getItem('vault');
        if (encryptedVault) {
            const encrypted = JSON.parse(encryptedVault);
            const decrypted = await this.decrypt(encrypted);
            this.vault = JSON.parse(decrypted);
        }
    }
    
    arrayToBase64(array) {
        return btoa(String.fromCharCode.apply(null, array));
    }
    
    base64ToArray(base64) {
        return new Uint8Array(atob(base64).split('').map(c => c.charCodeAt(0)));
    }
    
    async hashPassword(password) {
        const encoder = new TextEncoder();
        const data = encoder.encode(password);
        const hash = await crypto.subtle.digest('SHA-256', data);
        return this.arrayToBase64(new Uint8Array(hash));
    }
}

const pm = new PasswordManager();
```

## Technology Stack
- Web Crypto API (SubtleCrypto)
- AES-256-GCM encryption
- PBKDF2 key derivation
- localStorage (encrypted)

## Testing Checklist
- [ ] Master password required
- [ ] Encryption/decryption works
- [ ] Password generator creates strong passwords
- [ ] Strength meter accurate
- [ ] Vault persists securely
- [ ] Cannot access without master password

## Security Considerations
- Never store master password
- Use PBKDF2 with 100,000+ iterations
- Always encrypt before storing
- Clear clipboard after copying
- Auto-lock after inactivity

## License
MIT - [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library)
