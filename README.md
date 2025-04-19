# xabierland.github.io

Este es el repositorio de mi blog personal, [xabierland.github.io](https://xabierland.github.io/).

## Montar el blog en local

### Instalar Ruby

```bash
sudo apt install ruby-full build-essential
```

### Añadir Ruby a la variable PATH

```bash
echo 'export GEM_HOME="$HOME/.gems"' >> ~/.zshrc
echo 'export PATH="$HOME/.gems/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Instalar Jekyll

```bash
gem install jekyll bundle
```

### Instalar las dependencias de Ruby

```bash
bundle config set path 'vendor/bundle'
bundle install
```

### Arrancar el servidor

```bash
bundle exec jekyll serve
```
