# Mi Blog

Blog hecho con Jekyll y alojado gratis en GitHub Pages.

## Escribir un artículo nuevo
1. Crea `_posts/AAAA-MM-DD-titulo.md` (copia el de bienvenida como plantilla).
2. `git add . && git commit -m "Nuevo post" && git push`
3. En 1-2 minutos aparece en la web.

## Ver el blog en local (opcional)
    sudo apt install ruby-full build-essential
    gem install --user-install bundler
    bundle install
    bundle exec jekyll serve
Abre http://localhost:4000
