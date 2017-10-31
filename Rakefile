require 'yaml'

CONFIGURATIONS = YAML.load_file('configurations.yml')['configurations']
CONFIG_DIR = File.expand_path(File.dirname(__FILE__))
OUT_DIR = File.expand_path(File.join(File.dirname(__FILE__), "out"))

def mixins(name)
  CONFIGURATIONS[name]['mixins'].each do |mixin|
    `cp -rp #{CONFIG_DIR}/files/#{mixin}/* #{OUT_DIR}/#{name}/files`
  end
end

def wifi(name)
  config_path = File.join(OUT_DIR, name, "/files/etc/config/wireless")
  wlans = CONFIGURATIONS[name]['wlans']
  wlan_config = File.read(config_path)

  # TODO: hack for 11ac channels, make more generic
  fiveg_channels = {}
  fiveg_channels['42'] = [36, 40, 44, 48]
  fiveg_channels['58'] = [52, 56, 60, 64]
  fiveg_channels['106'] = [100, 104, 108, 112]
  fiveg_channels['122'] = [116, 120, 124, 128]
  fiveg_channels['138'] = [132, 136, 140, 144]
  fiveg_channels['155'] = [149, 153, 157, 161]

  wlans.each do |id, wlan|
    wlan_config.sub!("<wlan#{id}-macaddr>", wlan['bssid'].to_s)
    # 2.4 GHz
    if id == 1
        wlan_config.sub!("<radio#{id}-channel>", wlan['channel'].to_s)
    # 5 GHz
    else
        channels = fiveg_channels[wlan['channel'].to_s]
        (0..3).each do |chan|
            wlan_config.sub!("<radio#{id}-channel#{chan}>", channels[chan].to_s)
        end
    end
  end

  File.open(config_path, 'w') do |f|
    f.puts wlan_config
  end
end

def hostname(name)
  `sed -i "s/<hostname>/#{name}/g" #{OUT_DIR}/#{name}/files/etc/config/system`
  `sed -i "s/<hostname>/#{name}/g" #{OUT_DIR}/#{name}/files/etc/collectd.conf`
end

def ip(name)
  ip = CONFIGURATIONS[name]['ip']
  `sed -i "s/<ip>/#{ip}/g" #{OUT_DIR}/#{name}/files/etc/config/network`
end

def config(name)
  config = CONFIGURATIONS[name]['config']
  `cp #{File.join(CONFIG_DIR, "%s.configdiff" % config)} #{File.join(OUT_DIR, name, '.config')}`
end

def outdir(name)
  `mkdir -p #{File.join(OUT_DIR, name, 'files')}`
end

task :ap, [:name] do |t, args|
  outdir args.name
  mixins args.name
  hostname args.name
  wifi args.name
  ip args.name
  config args.name
end

task :all do
  aps = %W(ap-cf-f-l ap-cf-f-r) # Foaier
  aps += %W(ap-cf-mr-1 ap-cf-mr-2 ap-cf-mr-3 ap-cf-mr-4) # Main room
  aps += %W(ap-cf-sr-1 ap-cf-sr-2 ) # Small room
  aps += %W(ap-cf-ch ap-cf-qws) # Chillout and quiet workshop area
 
  # Workshop floor
  aps += %W(ap-ws-ws1 ap-ws-ws2) # workshop and overflow area
  aps += %W(ap-ws-noc) # NOC/Team room

  aps.each do |ap|
    Rake::Task["ap"].reenable
    Rake::Task["ap"].invoke ap
  end
end
