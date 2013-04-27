require 'pathname'
require 'config.rb'

timestamp=Time.now.strftime("%Y%m%d%H%M%S")

task :config do
    $projects.each { |name, project|
      puts "[#{name}]"
      puts "sshHost   = #{project["sshHost"]}"
      puts "sshUser   = #{project["sshUser"]}"
      puts "sshPort   = #{project["sshPort"]}"
      puts "mysqlUser = #{project["mysqlUser"]}"
      puts "mysqlPass = #{project["mysqlPass"]}"
      puts "mysqlDb   = #{project["mysqlDb"]}"
      puts "interval  = #{project["interval"]}"
    }
end

task :setup do
  $projects.each { |name, project|
    puts " > installing public key on #{name}"
    system("ssh-copy-id #{project["sshUser"]}@#{project["sshHost"]}")
  }
end

task :backup do
  
  $projects.each { |name, project|
    
    # Create dir if not exists
    FileUtils.mkpath "backup/#{name}"
    
    # Check last updated
    backups = Dir.glob("backup/#{name}/#{name}*")
    currentTime = Time.new
    interval = 999999999;
    backups.each { |backup|
      timeinfo = backup.scan(/([0-9]{4})([0-9]{2})([0-9]{2})([0-9]{2})([0-9]{2})([0-9]{2})/)
      backupTime = Time.local(timeinfo[0][0], timeinfo[0][1], timeinfo[0][2], timeinfo[0][3], timeinfo[0][4], timeinfo[0][5])
      interval = [interval, currentTime - backupTime].min;
      
      # Detele old archives
      if interval > project["expire"]
        File.delete(backup)
      end
    }
    
    if interval > project["interval"]
      # Backup db
      puts " > Backing up #{name}"
      system("ssh #{project["sshUser"]}@#{project["sshHost"]} \"mysqldump -u #{project["mysqlUser"]} --password='#{project["mysqlPass"]}' #{project["mysqlDb"]}\" |gzip > backup/#{name}/#{name}-#{timestamp}.sql.gz")
    end

  }
  
end
