require 'yaml'
require 'fileutils'

namespace :images do
  desc "Export docker images"
  task :export do
    include ImageConfig
    include CommonDirs

    images.each do |image|
      sh "docker pull #{image.name}"
      sh "docker save #{image.name} > #{tmp_dir(image.tar)}"
    end
  end

  desc "Package exported images"
  task :package do
    include DockerImagePackaging
    include ImageConfig
    include CommonDirs

    images.each do |image|
      Dir.mktmpdir do |dir|
        repackage_image_blobs(tmp_dir(image.tar), dir).tap do |blobs|
          blobs.each { |b| sh "bosh add blob #{b.target(dir)} #{b.prefix}" }
          create_package(image.package, blobs.map(&:path))
        end
      end
    end
  end
end

module CommonDirs
  def repo_dir
    File.expand_path("../", __FILE__)
  end

  def tmp_dir(path = "")
    File.join(repo_dir, 'tmp', path)
  end

  def packages_dir(path = "")
    File.join(repo_dir, 'packages', path)
  end
end

module ImageConfig
  def images
    @images ||= begin
      YAML.load_file(File.expand_path('../images.yml', __FILE__))
        .map! { |i| Image.new(i["image"], i["tag"]) }
    end
  end

  class Image
    def initialize(name, tag)
      @name = name
      @tag = tag
    end

    def name
      @name + ":" + @tag
    end

    def package
      name.gsub(/[\/\-\:\.]/, '_') + "_image"
    end

    def tar
      "#{package}.tgz"
    end
  end
end

module DockerImagePackaging
  PREFIXES = %w(docker_layers docker_images)

  class Blob
    attr_reader :source, :target, :prefix
    def initialize(source, target, prefix)
      @source = source
      @target = target.sub('.tgz', '') + '.tgz'
      @prefix = prefix
    end

    def target(dir = nil)
      dir ? File.join(dir, @target) : @target
    end

    def path
      "#{@prefix}/#{@target}"
    end
  end

  def repackage_image_blobs(image_tar, target_dir)
    Dir.chdir(target_dir) do
      sh "tar -xf #{image_tar}"

      blobs = Dir.glob("*/").map! { |d| Blob.new(d.chop, d.chop, PREFIXES[0]) }
      blobs << Blob.new('repositories', File.basename(image_tar), PREFIXES[1])

      package_blobs(blobs)
    end
  end

  def package_blobs(blobs)
    blobs.each { |b| sh "tar -zcf #{b.target} #{b.source}" }
  end

  def create_package(name, files)
    package_dir = File.expand_path("../packages/#{name}", __FILE__)
    FileUtils.mkdir_p package_dir
    spec = { "name" => name, "files" => files }
    IO.write(File.join(package_dir, 'spec'), spec.to_yaml)
    IO.write(File.join(package_dir, 'packaging'), packaging_script)
  end

  def packaging_script
    <<-END.gsub(/^ {6}/, '')
      set -e; set -u
      cd docker-layers
      for layer in *.tgz; do tar -xf "$layer"; rm "$layer"; done
      tar -xf ../docker-images/*.tgz
      tar -zcf image.tgz ./*
      cp -a image.tgz $BOSH_INSTALL_TARGET
    END
  end
end
