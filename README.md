EBRS SETUP.

NOTE: ruby v2.1.2 required

1. Clone application
	git clone https://github.com/HISMalawi/ebrs-touch.git ebrs_facility

2. git checkout optimisation

3. cd into config and copy all .example files to .yml

4. Edit database.yml with right mysql credentials
	
	username
	password

5. in settings.yml make changes to these paths to reflect like this
	barcodes_path: /var/www/ebrs_facility/barcodes/
    	certificates_path: /var/www/ebrs_facility/certificates/

	and also add a ":" at the end of this "line enable_printing_when_hq_is_offline"

6. run "bundle install" if you dont have ruby gems or "bundle install --local" if ruby gems already existing in you machine in the application folder

7. Run the following command in the application folder "./setup.sh production /var/www/metadata.sql" (Please note that you can either run it in development of production)

8. load the schema & metadata in this specific order. Note that you will load to the database you have initialised, either in production or development.

	mysql -uroot -proot ebrs_facility_2_0_development < db/schema.sql
	mysql -uroot -proot ebrs_facility_2_0_development < db/metadata.sql

9. Create admin user using the following command

	bundle exec rake db:seed

10. Launch the application using the following command

	rails s -p3000 -b0.0.0.0 
	or
	passenger start  -p3000

11. On the server configure nginx file to automatically start the application. example of the nginx block below:

	server { 
	   listen 3000; 
	   server_name ebrs_facility; 
	   passenger_enabled on; 
	   root /var/www/ebrs_facility/public; 
	}
