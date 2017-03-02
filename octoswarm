#!/bin/bash

SCRIPT_NAME="octoswarm"
STACK_CONFIG_PATH="$PWD/cluster.json"

usage(){
  echo "USAGE: $SCRIPT_NAME"
  echo 'Description: swarms of octos!'
  echo ''
  echo 'Arguments:'
  echo '  -h, --help        print this help text'
  echo '  -v, --version     print the version'
  echo '  -u, --upgrade     upgrade to the latest version of octoswarm'
  echo 'Environment:'
  echo '  DEBUG             print debug output'
}

version(){
  local directory
  directory="$(script_directory)"

  if [ -f "${directory}/VERSION" ]; then
    cat "${directory}/VERSION"
  else
    echo "unknown-version"
  fi
}

script_directory(){
  local source="${BASH_SOURCE[0]}"
  local dir=""

  while [ -h "$source" ]; do # resolve $source until the file is no longer a symlink
    dir="$( cd -P "$( dirname "$source" )" && pwd )"
    source="$(readlink "$source")"
    [[ $source != /* ]] && source="$dir/$source" # if $source was a relative symlink, we need to resolve it relative to the path where the symlink file was located
  done

  dir="$( cd -P "$( dirname "$source" )" && pwd )"

  echo "$dir"
}

fatal() {
  local message="$@"
  errecho "Error: $message"
  exit 1
}

errecho() {
  local message="$@"
  (>&2 echo -e "$message")
}

assert_cluster_json() {
  if [ ! -f "$STACK_CONFIG_PATH" ]; then
    fatal "octoswarm must be run in a folder with a cluster.json"
  fi
}

get_docker_version() {
  jq --raw-output '.octoswarm.dockerVersion' "$STACK_CONFIG_PATH"
}

get_binary_name() {
  local docker_version="$(get_docker_version)"
  echo "octoswarm-$docker_version"
}

get_whalebrew_image() {
  local docker_version="$(get_docker_version)"
  echo "octoblu/octoswarm:$docker_version"
}

assert_required_config() {
  local docker_version="$(get_docker_version)"
  if [ "$docker_version" == "null" ]; then
    fatal "Octoswarm docker version not specified, exiting."
  fi
}

main() {
  local upgrade
  case "$1" in
    -h | --help)
      usage
      exit 0
      ;;
    -v | --version)
      version
      exit 0
      ;;
    -u | --upgrade)
      upgrade='true'
      shift
      ;;
  esac

  local args="$@"
  assert_cluster_json
  assert_required_config
  local binary_name="$(get_binary_name)"
  local whalebrew_image="$(get_whalebrew_image)"
  if [ "$upgrade" == "true" ]; then
    echo "Uninstalling $binary_name from whalebrew"
    whalebrew uninstall "$whalebrew_image"
  fi
  if [ -z "$(which "$binary_name")" ]; then
    echo "Installing $binary_name from whalebrew"
    whalebrew install "$whalebrew_image" || fatal "Unable to install $whalebrew_image, cowardly refusing to do anything."
  fi
  $binary_name $args
}

main "$@"
