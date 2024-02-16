## Defaults
- `main.yml` : Default variables for this role
  - `default.php`
    - `php_enable_webserver` : Using PHP with web server. It'll process restart FPM & copy test file
    - `php_enable_php_fpm` : Use FPM
    - `php_use_managed_ini` : Use managed php.ini. Located in `templates/php.ini.j2`

## Tasks
- `main.yml` : The main list of tasks that the role executes.
  - Load variables
  - Setup APT `ppa:ondrej/php`
  - Defines variables
  - Execute `setup-Debian` or `setup-RedHat` if not enable `php_install_from_source`
    - Update APT
    - Install packages
  - General configuration for PHP
  - Config for APCu
  - Config for Opcache
  - Config for FPM
  - Redis & MongoDB will be installed by default