Souce for my blog + micro blog.

# To test on local machine
## Use hugo server
```
hugo server --bind 192.168.0.107 --baseURL=http://192.168.0.107:1313
```
where `192.168.xxx.xxx` is your local machine's ip address. This way you can open `http://192.168.0.107:1313` from another PC or mobile device to view your website.

## Use an http server for the generated static site
First, run
```
rm -rf public/* &&  hugo -D
```
Hugo generates the static site in `public/` folder. We need to remove `public/*` (except the first time you run `hugo`) before regenerating. `-D` is for building drafts.

Then, `cd` into `public/` folder and run
```
python3 -m http.server --bind 192.168.0.107 8080
```

For me the use of `baseURL` is very confusing in hugo's document. I set it to `'/'` in my `config.toml`. And I got the above local deployment procedure to work through several try and error. By simply following hugo documentation, I got different kinds of errors, such as:
1. If I only run `hugo server`, I can only visit the site from the server machine, but not other lan machine through ip address.
2. Without proper `baseURL` tweak, site templates cannot be rendered properly.