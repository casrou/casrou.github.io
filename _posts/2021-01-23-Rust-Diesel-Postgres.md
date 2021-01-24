---
layout: post
title: Minimal Diesel + Postgres Setup
---

Setup a minimal database connection from rust using [Diesel](http://diesel.rs/) and [Postgres](https://www.postgresql.org/).

If you run into any errors, check the "Errors" section at the end.

## Diesel
We follow the ["Getting Started" guide](http://diesel.rs/guides/getting-started/) from the diesel website.

```
cargo new --lib diesel_demo
cd diesel_demo
```

Add dependencies to the `Cargo.toml` file:
```
[dependencies]
diesel = { version = "1.4.4", features = ["postgres"] }
dotenv = "0.15.0"
```

Now, we want to install Diesel CLI.

### Windows (not working, go to next section)
(I left my incomplete progress with the installation of Diesel CLI on Windows here in case it could help others more determined to get it to work. I would recommend just using WSL instead..)

Try to run the command:

`cargo install diesel_cli --no-default-features --features postgres`

I ran into an error (`error: linking with 'link.exe' failed: exit code: 1181`) when installing on Windows. 

To fix it, I had to install PostgreSQL from [EDB](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads), add an environment variable `PQ_LIB_DIR` and set it to `C:\Program Files\PostgreSQL\13\lib`. 

(You might also have to add `C:\Program Files\PostgreSQL\13\bin` and `C:\Program Files\PostgreSQL\13\lib` to the `PATH` environment variable, but it wasn't necessary for me).

Remember, to change the path to where you've installed PostgreSQL and **restart your console before re-running the installation command**.

Before I fixed the error, I ran the installation command with the `--verbose` argument to get a longer error message. I will add parts of it here, in case it could help others find the fix:

```
error: linking with `link.exe` failed: exit code: 1181
[omitted]
  = note: LINK : fatal error LNK1181: cannot open input file 'libpq.lib'


error: aborting due to previous error

error: failed to compile `diesel_cli v1.4.1`, intermediate artifacts can be found at `C:\Users\caspe\AppData\Local\Temp\cargo-installpOaveg`

Caused by:
  could not compile `diesel_cli`

Caused by:
  process didn't exit successfully: `rustc --crate-name diesel
  [omitted]
  ` (exit code: 1)
```

At this point, Diesel seemed to be successfully installed. However, when attempting to run `diesel` in Powershell, nothing happens. If we instead run `diesel` in Command Prompt, errors appear about missing `.dll`'s. More specifically these three `.dll`'s:
- `libcrypto-1_1-x64.dll`
- `libssl-1_1-x64.dll`
- `libintl-8.dll`

You may be able to fix it and use `diesel` on Windows, but I gave up at this point and chose to use WSL instead.

### Ubuntu (WSL)

- [Install PostgreSQL](https://docs.microsoft.com/en-us/windows/wsl/tutorials/wsl-database#install-postgresql)

- Run `sudo apt install build-essential` to avoid "linker cc"-errors.

- Run `sudo apt install libpq-dev` to avoid "/usr/bin/ld: cannot find -lpq"-errors.

Install Diesel CLI with the same command as before:

`cargo install diesel_cli --no-default-features --features postgres`

You should now (finally!) be able to run `diesel --version` and see the version of Diesel CLI get printed to the console.

`echo DATABASE_URL=postgres://postgres:password@localhost/diesel_demo >
.env`

Then, run `diesel setup`.

Even though I changed the password in the above command to the one used in the "Install PostgreSQL" guide, I was met with the following errors:

```
Creating database: diesel_demo
FATAL:  password authentication failed for user "postgres"
FATAL:  password authentication failed for user "postgres"
```
To fix this, I opened the PostgreSQL command prompt with `sudo -u postgres psql`. Then, I was able to change the password with `\password postgres`. Exit with `\q` and run `diesel setup` again.

Now, we should get just the message `Creating database: diesel_demo` without any errors.

We continue with the "Getting Started" guide.

Add a migration with `diesel migration generate create_posts`

Change the `up.sql` file found in the `migrations/*timestamp*_create_posts` folder. It should contain the following:
```
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR NOT NULL,
  body TEXT NOT NULL,
  published BOOLEAN NOT NULL DEFAULT 'f'
)
```
And change the `down.sql` file from the same folder, such that it contains:
```
DROP TABLE posts
```
Now, we apply the migration with `diesel migration run`.

## Rust
Create or change the following files:

`src/lib.rs`
{% gist 9149edcbada0f7f52fd9fed590789b20 %}

`src/models.rs`
{% gist 2bce774e72849ca49866ff6dae19aec5 %}

`src/bin/write_post.rs`
{% gist 22007809f7f8c9d88409ae13a9ccf643 %}

Run the last file with `cargo run --bin write_post`.
Create a post with a title and content. For example:
```
   Compiling diesel-demo v0.1.0 (/mnt/c/Dev/diesel-demo)
    Finished dev [unoptimized + debuginfo] target(s) in 18.34s
     Running `target/debug/write_post`

What would you like your title to be?
Example Title

Ok! Let's write Example Title (Press CTRL+D when finished)

This is an example post about birds. They sit in trees. Eat worms. And they can fly!
Really - they soar through the wind.

That is all. Birds are cool.
```

Now create the `src/bin/publish_post.rs` file:
{% gist 4905991397b8b7ec68059db84fd21210 %}

Use it to publish the previous draft post:
```
cargo run --bin publish_post 1

   Compiling diesel-demo v0.1.0 (/mnt/c/Dev/diesel-demo)
    Finished dev [unoptimized + debuginfo] target(s) in 10.30s
     Running `target/debug/publish_post 1`

Published post Example Title
```

Create the file `src/bin/show_posts.rs`:
{% gist c41b624430466303fbbdd07f363f7e71 %}

Show all posts with `cargo run --bin show_posts`.

To also delete posts, we create the file `delete_post.rs`:
{% gist 01a84daab31211dd7040ec28477b3cb4}

Delete a post by its title by running `cargo run --bin delete_post "Example Title"`. Your output should look like this:
```
   Compiling diesel-demo v0.1.0 (/mnt/c/Dev/diesel-demo)
    Finished dev [unoptimized + debuginfo] target(s) in 14.62s
     Running `target/debug/delete_post 'Example Title'`
Deleted 1 posts
```

Now, we have set up a simple connection from Rust to PostgreSQL that supports the basic CRUD operations. In the next post, we will create a web API with Actix Web and Diesel containerized with Docker.

## Errors
### error[E0463]: can't find crate for `diesel`
Be sure you added the dependencies to the `Cargo.toml` file.