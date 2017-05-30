InfluxDB::Rails.configure do |config|
  config.influxdb_database = "mydb"
  config.influxdb_username = "vlad"
  config.influxdb_password = "vlad"
  config.influxdb_hosts    = ["localhost"]
  config.influxdb_port     = 8086

  # config.series_name_for_controller_runtimes = "rails.controller"
  # config.series_name_for_view_runtimes       = "rails.view"
  # config.series_name_for_db_runtimes         = "rails.db"
end
begin
# Create client
influxdb = InfluxDB::Client.new("platform_dev",
                                user_name: 'vlad',
                                password: 'vlad',
                                port: 8086,
                                host: "localhost",
                                time_precision: 's')

# Test Data
data1 = {
    values: { duration: 30 },
    tags: { user_id: 'friendly_id_1' },
    timestamp: Time.now.to_i
}

data2 = {
    values: { duration: 3 },
    tags: { user_id: 'friendly_id_2' },
    timestamp: Time.now.to_i
}


data3 = {
    values: { duration: 5 },
    tags: { user_id: 'friendly_id_1' },
    timestamp: (Time.now + 1.hour).to_i
}

data4 = {
    values: { duration: 10 },
    tags: { user_id: 'friendly_id_2' },
    timestamp: (Time.now + 1.hour).to_i
}

# Set data
[data1, data2, data3, data4].each do |d|
  influxdb.write_point("song8", d)
end



influxdb.query ('select sum("duration") from "song8" where '+ "time >= #{ convert_time(Time.now - 1.hour) } and time <= #{ convert_time(Time.now + 1.hours) } group by time(1h)")

def convert_time(time)
  # 1 nanosecond = 1/1,000,000,000 second
  time.to_i * 1_000_000_000
end

def extract_from_values(result, key)
  result[0]['values'].map{ |p| p[key] }
end

=begin
insert cpu,host=serverA value=0.64,a=1
insert table_name,tag_1=1,tag_2="tag_2" field_1=1,field_2="field_2"
Tags - optional
Fields - required

select * from table_name where tag_2=~ /tag_2/
select * from table_name where tag_2="tag_2"

From: https://docs.influxdata.com/influxdb/v1.2/concepts/schema_and_data_layout/

* Tags are indexed and fields are not indexed.
* Store data in tags if they’re commonly-queried meta data
* Store data in tags if you plan to use them with GROUP BY()
* !!! Store data in fields if you need them to be something other than a string - tag values are always interpreted as strings
* Store data in fields if you plan to use them with an InfluxQL function


Queries: https://docs.influxdata.com/influxdb/v1.2/query_language/
=end
