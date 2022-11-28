
# bundle_gems (OBS source service)

[![Build Status](https://travis-ci.org/openSUSE/obs-service-bundle_gems.svg?branch=master)](https://travis-ci.org/openSUSE/obs-service-bundle_gems)

This service is useful for Rails and similar applications using a [Bundler]() `Gemfile`.

Configured correctly it will:

* Read your source code and figure out dependent gems
* Add those gems to the rpm .spec file

# Usage

## spec strategy (default)

* You need a `Gemfile` and `Gemfile.lock` among your sources. If you have a tarball, consider using
  the [extract_file](https://github.com/openSUSE/obs-service-extract_file) service to extract them.
  
* If your tarball is created from a git repository using the [tar_scm](https://github.com/openSUSE/obs-service-tar_scm) source service, use the following parameters to tar_scm to extract the files:

```xml
  <param name="extract">Gemfile</param>
  <param name="extract">Gemfile.lock</param>
```

* You need to mark the spec file with a special comment block, after the last sources.

```
Source2: somefile.tar.gz
# From here populated by obs-service-bundle_gems
### GEMS START
### GEMS END
```

* Once there is a tarball, you configure the bundle_gems service.

```xml
<service name="bundle_gems"/>
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
...
```

Configure a service to retrieve those files:

```xml
  <service name="download_files"/>
```

* The resulting `_service` file would look like:

```xml
<services>
  <service name="tar_scm">
    <param name="versionformat">15.0.git%cd.%h</param>
    <param name="url">git://github.com/openSUSE/software-o-o.git</param>
    <param name="scm">git</param>
    <param name="extract">Gemfile</param>
    <param name="extract">Gemfile.lock</param>
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

* Include Ruby and Bundler as requirement:

```
BuildRequires: rubygem(%{rb_default_ruby_abi}:bundler)
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

## cpio mode
Compared to the default spec strategy mode, the cpio strategy mode makes use of bundler to create a ``vendor.obscpio`` file.
This has the advantage that you can also use gems from sources other than rubygems.org and nothing gets written to your spec file (you don't need the ```# GEMS START``` marker).
Most of the description from the spec strategy mode (default) applies as well in this mode.
However, you need to explictly set the strategy to `cpio` in your service file with:

```xml
<service name="bundle_gems">
  <param name="strategy">cpio</param>
</service>
```

The vendor obscpio gets automatically unpacked during build. The gems are located under ``rpmbuild/SOURCES/vendor/cache`` in your build environment.

### Specific Ruby interpreter version (using rbenv)
You can choose if needed the ruby version used to install the gems packed in the ``vendor.obscpio`` file:
```xml
<service name="bundle_gems">
  <param name="strategy">cpio</param>
  <param name="ruby-version">2.7.6</param>
</service>
```

It can be useful to build gems that have strong dependencies on the ruby interpreter (ie ``nokogiri``). The specified version is installed by ``rbenv`` (if not already present).
``rbenv`` and ``ruby-build`` are required.

# Authors

* Duncan Mac-Vicar P. <dmacvicar@suse.de>
* Ludwig Nussel <lnussel@suse.de>

# License

The code is licensed under the GPLv2 or later.
