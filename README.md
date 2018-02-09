
# bundle_gems (OBS source service)

This service is useful for Rails and similar applications using a [Bundler]() `Gemfile`.

Configured correctly it will:

* Read your source code and figure out dependent gems
* Add those gems to the rpm .spec file

# Usage

* You need a tarball first. Either your application is in a tarball, or you create one
  with [tar_scm]() source service from your application git repository.

* You need to mark the spec file with a special comment block, after the last sources.

```
Source2: somefile.tar.gz
# From here populated by obs-service-bundle_gems
### GEMS START
### GEMS END
```

* Once there is a tarball, you configure the bundle_gems service.

```xml
<services>
  <service name="tar_scm">
    <param name="versionformat">15.0.git%cd.%h</param>
    <param name="url">git://github.com/openSUSE/software-o-o.git</param>
    <param name="scm">git</param>
  </service>
  <service name="bundle_gems"/>
  <service name="download_files"/>
  <service name="recompress">
    <param name="compression">gz</param>
    <param name="file">*.tar</param>
  </service>
  <service name="set_version">
  </service>
</services>
```

* As after the `bundle_gems` service, the gems will be listed in the rpm spec as a URL, you can configure the `download_files` gem to retrieve them.

```
# From here populated by obs-service-bundle_gems
### GEMS START
Source100: https://rubygems.org/downloads/actioncable-5.1.4.gem
Source101: https://rubygems.org/downloads/actionmailer-5.1.4.gem
Source102: https://rubygems.org/downloads/actionpack-5.1.4.gem
Source103: https://rubygems.org/downloads/actionview-5.1.4.gem
Source104: https://rubygems.org/downloads/activejob-5.1.4.gem
Source105: https://rubygems.org/downloads/activemodel-5.1.4.gem
```

* Include Ruby and Bundler as requirement:

```
Source099: rubygem(%{rb_default_ruby_abi}:bundler)
```

* Install the gems in the vendor/cache in order to run tasks needed during build:

```
mkdir -p vendor/cache
cp %{_sourcedir}/*.gem vendor/cache

%build
gem="gem.%{rb_default_ruby_suffix}"
bundle="bundle.%{rb_default_ruby_suffix}"
export GEM_HOME=$PWD/vendor GEM_PATH=$PWD/vendor PATH=$PWD/vendor/bin:$PATH
$gem install vendor/cache/bundle*.gem
$bundle config build.nokogiri --use-system-libraries
$bundle --local --deployment --with production
```

# Authors

* Duncan Mac-Vicar P. <dmacvicar@suse.de>
* Ludwig Nussel <lnussel@suse.de>

# License

The code is licensed under the GPLv2 or later.

