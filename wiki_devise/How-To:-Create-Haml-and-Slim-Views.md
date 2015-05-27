The Haml/Slim view generators were removed from Devise 1.2. Here is a tutorial how to create Haml/Slim views with Devise 1.2 or later.

## Summary

```sh
rails generate devise:views

gem install html2haml
for file in app/views/devise/**/*.erb; do html2haml -e $file ${file%erb}haml && rm $file; done

gem install html2slim
for file in app/views/devise/**/*.erb; do erb2slim $file ${file%erb}slim && rm $file; done
```

### On Windows

```cmd
rails generate devise:views

gem install html2haml
for /r app\views\devise %I in (*) do html2haml -e "%I" "%~dpnI.haml"
del /f /s /q app\views\devise\*.erb

gem install html2slim
for /r app\views\devise %I in (*) do erb2slim "%I" "%~dpnI.slim"
del /f /s /q app\views\devise\*.erb
```

## Explanation

### Create Haml views

First you need run the generator to create the devise ERB files.

```sh
rails generate devise:views
```

Now we need a tool called '[html2haml](https://github.com/haml/html2haml)' which was formely a part of the '[haml](https://github.com/haml/haml)' gem but is now separate.

```sh
gem install html2haml
```

Now for each erb file, generate a HAML copy.

```sh
find ./app/views/devise/** -name \*.erb -print | sed 'p;s/.erb$/.haml/' | xargs -n2 html2haml
```

After confirming that everything went as planned, delete the erb counterparts.

```sh
rm app/views/devise/**/*.erb
```

### Create Slim views

We use a gem called '[html2slim](https://github.com/slim-template/html2slim)' to create the Slim-views.

```sh
gem install html2slim
```

This package include a tool called `erb2slim` which can convert erb file to slim recursively.
Option `-d` for delete the erb file after the convert finished.

```sh
erb2slim -d DIR
```