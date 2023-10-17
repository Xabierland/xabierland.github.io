# GPG Cheatsheet

## Generate a new key

```bash
gpg --gen-key
gpg --full-gen-key
```

## List keys

```bash
gpg -k  # public keys
gpg -K  # private keys
```

## Export keys

```bash
gpg -o key.asc --armor --export <KEY ID>
```

## Import keys

```bash
gpg --import key.asc
```

## Trusting a key

```bash
gpg --edit-key <KEY ID>
gpg> trust
gpg> save
```

## Using a keyserver
