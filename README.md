## install-picrate

To create and run `processing` sketches on the `raspberryPI` using `ruby` and the `geanyIDE`, you only need to install the `picrate` gem (and its dependencies ie `jruby`).
This is super using the provided `Rakefile`:-

Which installs jruby, configures gem home and geany, and installs picrate gem

Simple clone or download this repo

```bash
cd install-picrate
rake # rake --tasks to see Tasks
```

Some lines in script require sudo authority (eg to write to `/opt` folder), so you will be asked for password.

Installs latest jruby etc....
