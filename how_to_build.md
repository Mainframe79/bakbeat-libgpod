	•	Homebrew deps (glib, libplist, sqlite, pkg-config, maybe mono if needed)
meson setup builddir --prefix=$PWD/_install -Dpython=disabled -Dtest=false
meson compile -C builddir
meson install -C builddir