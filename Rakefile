namespace :packages do
  desc "Update docker image packages"
  task :update_docker_images, [:manifest]  do |t, args|
    require 'yaml'
    manifest = YAML.load_file(File.expand_path(args[:manifest], __FILE__))
    services = manifest["properties"]["broker"]["services"]
    services.map! do |service|
      service["plans"].map! do |plan|
        OpenStruct.new(
          plan["container"].select do |k|
            %w(image tag).include? k
          end
        )
      end
    end.flatten!.uniq! { |s| s.image + s.tag }

#    print services.map(&:to_h).to_yaml
    services.each do |service|
      image = service.image + ":" + service.tag
      name = service.image.split('/')[1] + "-image"
      puts "Adding/updating package: #{name}"
      sh "bosh-gen package #{name} --docker-image #{image}"
    end
  end
end
