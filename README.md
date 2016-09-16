## Creating a detect script

1. Create a new folder for the buildpack:
  ```
  mkdir ~/custom_buildpack
  cd ~/custom_buildpack
  mkdir bin
  ```

2. Create the file `bin/detect` with the following content:
  ```
  #!/usr/bin/env bash
  # bin/detect <build-dir>

  if [ -f $1/Staticfile ]; then
    echo "staticfile" && exit 0
  else
    echo "no" && exit 1
  fi
  ``` 
## Creating a compile script

1. Create the file `bin/compile` with the following content:
  ```
  #!/usr/bin/env bash
  # bin/compile <build-dir> <cache-dir>

  set -e            # fail fast

  status() {
    echo "-----> $*"
  }

  # Configure directories
  build_dir=$1
  cache_dir=$2

  status "Moving application content into public folder"
  cd $build_dir
  mkdir public
  shopt -s extglob dotglob
  mv !(public) public 

  status "Installing static web server"
  wget -q -O static-fileserver  https://github.com/s-matyukevich/static-fileserver/blob/master/bin/static-fileserver?raw=true 
  chmod +x static-fileserver

  status "Creating boot script"
  cat > boot.sh << \EOF
  setsid $HOME/static-fileserver -p $PORT -d $HOME/public > out 2>out.err
  EOF
  chmod +x boot.sh

  echo "static"
  exit 0

  ```
## Creating a release script

1. Create the file `bin/release` with the following content:
  ```
  #!/usr/bin/env bash

  cat << EOF
  default_process_types:
    web: sh boot.sh 
  EOF
  ```
## Package and upload the buildpack

1. Install `zip`:
  ```
  sudo apt-get install zip
  ```

2. Add executable permissions:
  ```
  chmod +x ~/custom_buildpack/bin/*
  ```

3. Package the buildpack:
  ```
  cd ~
  zip -r custom_buildpack.zip custom_buildpack/
  ```

4. Login to Cloud Foundry (the default user/password in the manifest were `admin/admin`):
  ```
  cf login
  ```

5. Upload the buildpack:
  ```
  cf create-buildpack custom_buildpack custom_buildpack.zip 1
  ```

6. See the available buildpacks:
  ```
  cf buildpacks
  ```
## Verify the buildpack

1. Download a static application:
  ```
  cd ~
  git clone https://github.com/s-matyukevich/iot-dashboard
  ``` 

2. Add `Staticfile` to the appliction to make the buildpack recognize it:
  ```
  cd ~/iot-dashboard
  touch Staticfile
  ```

3. Deploy the application:
  ```
  cf push static
  ```
