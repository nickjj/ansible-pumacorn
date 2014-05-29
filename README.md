## What is ansible-pumacorn?

It is an [ansible](http://www.ansible.com/home) role to manage either a puma or unicorn rails process.

### What problem does it solve and why is it useful?

It is usually a good idea to create a service out of your rails application. This allows you to easily manage it in the future without having to worry about things like sourcing environments because the service script would handle all of the dirty work for you.

It gives you access to these commands after it's setup:

- `sudo service yourapp start`
- `sudo service yourapp stop`
- `sudo service yourapp restart`
- `sudo service yourapp reload`
- `sudo service yourapp status`

#### What's the difference between restart and reload?

**Restart** will have a small amount of downtime because it will kill off the master process for whatever web server you're using and fully restart it.

**Reload** will apply new code changes without downtime and the web server will cycle its workers automatically. Keep in mind if you do a reload then you will have (2) versions of your application running at the same time. If your application code is not setup to work with both versions then you will need to do a restart instead.

## Role variables

Below is a list of default values along with a description of what they do.

```
# Are you using puma or unicorn? This would be the binary name.
pumacorn_server: puma

# The path to the config file.
pumacorn_config_path: config/{{ pumacorn_server }}.rb

# Which signal type is used to stop the service?
pumacorn_stop_signal: QUIT

# Which signal type is used to do a 0 downtime reload?
pumacorn_reload_signal: USR1 # USR2 for unicorn

# Should a restart be forced?
pumacorn_force_restart: false
```

## Example playbook

For the sake of this example let's assume you have a group called **app** and you have a typical `site.yml` file.

To use this role edit your `site.yml` file to look something like this:

```
---
- name: ensure app servers are configured
- hosts: app

  roles:
    # Insert other roles here to provision the server before managing the rails process.
    - { role: nickjj.pumacorn, tags: pumacorn }
```

Let's say you want to edit a few defaults, you can do this by opening or creating `group_vars/all.yml` which is located relative to your `inventory` directory and then making it look something like this:

```
---
pumacorn_server: unicorn
pumacorn_reload_signal: USR2
```

#### Does it automatically restart a crashed process?

Nope, init.d scripts don't do that. What you get is a service that starts on bootup, restarts when a migration was ran and reloads with no downtime when no migrations were ran. If you want the process to be monitored you will want to hook up something like [monit](http://mmonit.com/monit), [runit](http://smarden.org/runit) or [god](http://godrb.com/).

I personally prefer monit because the API for adding new things to monitor is very nice and it optionally exposes a web front end.

## Installation

`$ ansible-galaxy install nickjj.pumacorn`

## Dependencies

It depends on the [nickjj.rails](https://github.com/nickjj/ansible-rails) role. This role must be ran before pumacorn.

## Requirements

Tested on ubuntu 12.04 LTS but it should work on other versions that are similar.

## License

MIT