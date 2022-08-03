# laravel-ci
A self-contained docker image for running CI on Laravel projects

Should require minimal effort to test Laravel projects. Comes with in-built myqsl 8, and most common PHP extensions.

Default environment is set to use local database for testing, send email through the "array" driver and use the "sync" queue driver

Running the `prepare-env` script will autoconfigure the database, build composer and npm, set an app key (if one isn't already set), and attempt to build front-end assets.
