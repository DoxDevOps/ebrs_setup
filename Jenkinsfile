pipeline {
  agent any
  stages {
    stage('Initializing') {
      steps {
        echo 'Initializing pipeline'
      }
    }

    stage('Fetching EDRS Repo') {
      steps {
        echo 'Starting to fetch EDRS application from GitHub'
        echo 'Checking if edrs_facility folder already exists'
        sh '[ -d "edrs_facility" ] && echo "edrs_facility already cloned." || git clone https://github.com/HISMalawi/edrs_dc.git edrs_facility'
        echo 'Giving all users access to the folder'
        sh 'chmod 777 $WORKSPACE/edrs_facility'
        echo 'Changing to couchdb-removed branch'
        sh 'cd $WORKSPACE/edrs_facility && git checkout couchdb-removed'
        echo 'Pulling Latest Changes for couchdb-removed branch'
        sh '''#cd $WORKSPACE/edrs_facility && git checkout $(git describe --tags `git rev-list --tags --max-count=1`)
cd $WORKSPACE/edrs_facility && git pull origin couchdb-removed'''
      }
    }

    stage('Configuring EDRS') {
      parallel {
        stage('Fetching touchscreentoolkit') {
          steps {
            echo 'Checking if touchscreentoolkit already exists'
            sh '[ -d "$WORKSPACE/edrs_facility/public/touchscreentoolkit" ] && echo "touchscreentoolkit already cloned." || git clone https://github.com/HISMalawi/touchscreentoolkit.git $WORKSPACE/edrs_facility/public/touchscreentoolkit'
            echo 'Pulling to check for latest changes'
            sh 'cd $WORKSPACE/edrs_facility/public/touchscreentoolkit && git fetch --tags -f #&& git checkout $(git describe --tags `git rev-list --tags --max-count=1`)'
            echo 'all changes up-to-date'
          }
        }

        stage('Fetching couchdb-dump') {
          steps {
            echo 'Checking if couchdb-dump already exists'
            sh '[ -d "$WORKSPACE/edrs_facility/bin/couchdb-dump" ] && echo "couchdb-dump already cloned." || git clone https://github.com/danielebailo/couchdb-dump.git $WORKSPACE/edrs_facility/bin/couchdb-dump'
            echo 'Pulling to check latest changes'
            sh 'cd $WORKSPACE/edrs_facility/bin/couchdb-dump && git pull'
            echo 'Renaming couchdb-dump.sh to couchdb-backup.sh'
            sh '[ -f "$WORKSPACE/edrs_facility/bin/couchdb-dump/couchdb-backup.sh" ] && echo "couchdb-backup.sh already exists." || mv $WORKSPACE/edrs_facility/bin/couchdb-dump/couchdb-dump.sh $WORKSPACE/edrs_facility/bin/couchdb-dump/couchdb-backup.sh'
          }
        }

        stage('Copying .example files to .yml files') {
          steps {
            echo 'Changes taking place inside config folder'
            sh '[ -f "$WORKSPACE/edrs_facility/config/couchdb.yml" ] && echo "couchdb.yml already exists." || cp $WORKSPACE/edrs_facility/config/couchdb.yml.example $WORKSPACE/edrs_facility/config/couchdb.yml'
            echo 'Editing couchdb.yml file [replacing dc with facility]'
            sh 'sed -i \'s/dc/facility/\' $WORKSPACE/edrs_facility/config/couchdb.yml'
            sh '[ -f "$WORKSPACE/edrs_facility/config/database.yml" ] && echo "database.yml already exists." || cp $WORKSPACE/edrs_facility/config/database.yml.example $WORKSPACE/edrs_facility/config/database.yml'
            echo 'Editing database.yml file [replacing edrs_fc with edrs_fc'
            sh 'sed -i \'s/edrs_dc/edrs_fc/\' $WORKSPACE/edrs_facility/config/database.yml'
            sh '[ -f "$WORKSPACE/edrs_facility/config/db_mapping.yml" ] && echo "db_mapping.yml already exists." || cp $WORKSPACE/edrs_facility/config/db_mapping.yml.example $WORKSPACE/edrs_facility/config/db_mapping.yml'
            sh '[ -f "$WORKSPACE/edrs_facility/config/elasticsearchsetting.yml" ] && echo "elasticsearchsetting.yml already exists." || cp $WORKSPACE/edrs_facility/config/elasticsearchsetting.yml.example $WORKSPACE/edrs_facility/config/elasticsearchsetting.yml'
            sh '[ -f "$WORKSPACE/edrs_facility/config/secrets.yml" ] && echo "secrets.yml already exists." || cp $WORKSPACE/edrs_facility/config/secrets.yml.example $WORKSPACE/edrs_facility/config/secrets.yml'
            sh '[ -f "$WORKSPACE/edrs_facility/config/settings.yml" ] && echo "settings.yml already exists." || cp $WORKSPACE/edrs_facility/config/settings.yml.example $WORKSPACE/edrs_facility/config/settings.yml'
            echo 'Editing settings.yml'
            sh 'sed -i \'s/\\/home\\/usr\\/barcodes\\//\\/var\\/www\\/edrs_facility\\/config\\//; s/\\/home\\/usr\\/certificates\\//\\/var\\/www\\/edrs_facility\\/config\\//; s/\\/home\\/usr\\/dispatch\\//\\/var\\/www\\/edrs_facility\\/config\\//; s/\\/home\\/usr\\/qrcodes\\//\\/var\\/www\\/edrs_facility\\/config\\//; s/site_type\\: dc/site_type\\: facility/\' $WORKSPACE/edrs_facility/config/settings.yml'
            sh '[ -f "$WORKSPACE/edrs_facility/config/sync_settings.yml" ] && echo "sync_settings.yml already exists." || cp $WORKSPACE/edrs_facility/config/sync_settings.yml.example $WORKSPACE/edrs_facility/config/sync_settings.yml'
          }
        }

      }
    }

    stage('Shipping to remote server') {
      steps {
        sh '''#OpsUsers Server
#rsync -a $WORKSPACE/edrs_facility opsuser@10.44.0.52:/home/opsuser

#python3 edrs_shipping.py'''
      }
    }

    stage('Remote Server Configuration') {
      parallel {
        stage('Remote Server Configuration') {
          steps {
            echo 'Editng District id and Facility Code'
            sh '''#OpsUser
#ssh opsuser@10.44.0.52 "sed -i \'s/facility_code\\:/facility_code\\: 3333/; s/district_code\\:/district_code\\: DV3/\' /home/opsuser/edrs_facility/config/settings.yml"

#Ntchisi
#ssh meduser@10.41.150.10 "sed -i \'s/facility_code\\:/facility_code\\: 1210/; s/district_code\\:/district_code\\: NS/\' /var/www/edrs_facility/config/settings.yml"
#ssh meduser@10.41.150.10 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"

#Chiradzulu
#ssh nrb-admin@10.41.150.10 "sed -i \'s/facility_code\\:/facility_code\\: 2801/; s/district_code\\:/district_code\\: CZ/\' /var/www/edrs_facility/config/settings.yml"

#Mwanza
#ssh meduser@10.43.113.9 "sed -i \'s/facility_code\\:/facility_code\\: 3801/; s/district_code\\:/district_code\\: MN/\' /var/www/edrs_facility/config/settings.yml"
#ssh meduser@10.43.113.9 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"

#Nkhotakota
#ssh meduser@10.40.8.4 "sed -i \'s/facility_code\\:/facility_code\\: 1111/; s/district_code\\:/district_code\\: KK/\' /var/www/edrs_facility/config/settings.yml"
#ssh meduser@10.40.8.4 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"

#Salima
#ssh nrb-admin@10.41.154.4 "sed -i \'s/facility_code\\:/facility_code\\: 1415/; s/district_code\\:/district_code\\: SA/\' /var/www/edrs_facility/config/settings.yml"
#ssh nrb-admin@10.41.154.4 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"

#Bwaila
#ssh nrb-admin@10.41.5.20 "sed -i \'s/facility_code\\:/facility_code\\: 1503/; s/district_code\\:/district_code\\: LL/\' /var/www/edrs_facility/config/settings.yml"
#ssh nrb-admin@10.41.5.20 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"

#Nsanje
#ssh meduser@10.43.136.9 "sed -i \'s/facility_code\\:/facility_code\\: 3409/; s/district_code\\:/district_code\\: NE/\' /var/www/edrs_facility/config/settings.yml"
#ssh meduser@10.43.136.9 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"

#Phalombe
#ssh meduser@10.43.156.9 "sed -i \'s/facility_code\\:/facility_code\\: 3512/; s/district_code\\:/district_code\\: PE/\' /var/www/edrs_facility/config/settings.yml"
#ssh meduser@10.43.156.9 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"


#Rumphi
#ssh ebrs_server@10.2.12.201 "sed -i \'s/facility_code\\:/facility_code\\: 417/; s/district_code\\:/district_code\\: RU/\' /var/www/edrs_facility/config/settings.yml"
#ssh ebrs_server@10.2.12.201 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"

#Karonga
#ssh ebrs_server@10.40.24.20 "sed -i \'s/facility_code\\:/facility_code\\: 206/; s/district_code\\:/district_code\\: KA/\' /var/www/edrs_facility/config/settings.yml"
#ssh ebrs_server@10.40.24.20 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"

#KCH
#ssh nrb-admin@10.40.2.8 "sed -i \'s/facility_code\\:/facility_code\\: 1519/; s/district_code\\:/district_code\\: LL/\' /var/www/edrs_facility/config/settings.yml"
#ssh nrb-admin@10.40.2.8 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"

#Balaka
#ssh meduser@10.43.156.9 "sed -i \'s/facility_code\\:/facility_code\\: 3601/; s/district_code\\:/district_code\\: BLK/\' /var/www/edrs_facility/config/settings.yml"
#ssh meduser@10.43.156.9 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"

#Mzimba
#ssh ebrs_server@10.40.13.4 "sed -i \'s/facility_code\\:/facility_code\\: 532/; s/district_code\\:/district_code\\: MZ/\' /var/www/edrs_facility/config/settings.yml"
#ssh ebrs_server@10.40.13.4 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"

#Dedza
#ssh nrb-admin@10.41.84.10 "sed -i \'s/facility_code\\:/facility_code\\: 1707/; s/district_code\\:/district_code\\: DZ/\' /var/www/edrs_facility/config/settings.yml"
#ssh nrb-admin@10.41.84.10 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"

#Mchinji
#ssh meduser@10.41.152.10 "sed -i \'s/facility_code\\:/facility_code\\: 1609/; s/district_code\\:/district_code\\: MC/\' /var/www/edrs_facility/config/settings.yml"
#ssh meduser@10.41.152.10 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"

#Likoma
#ssh ebrsv2@10.40.29.20 "sed -i \'s/facility_code\\:/facility_code\\: 601/; s/district_code\\:/district_code\\: LA/\' /var/www/edrs_facility/config/settings.yml"
#ssh ebrsv2@10.40.29.20 "sed -i \'s/password\\: password/password\\: ebrs.root/\' /var/www/edrs_facility/config/database.yml"'''
          }
        }

        stage('Shipping ruby gems') {
          steps {
            sh '''#Ntchisi
#rsync -a /var/lib/jenkins/sourcegems.tgz meduser@10.41.150.10:/var/www/edrs_facility

'''
          }
        }

      }
    }

    stage('Extracting Shipped Ruby Gems') {
      steps {
        sh '''#Salima
#ssh nrb-admin@10.41.154.4 \'cd /var/www/edrs_facility && gemdir=`gem env | grep "\\- INSTALLATION DIRECTORY" | awk -F \': \' {\'print $2\'}`
#ssh nrb-admin@10.41.154.4 \'cd /var/www/edrs_facility && mkdir -p $gemdir\'
#ssh nrb-admin@10.41.154.4 \'cd /var/www/edrs_facility && tar xvfz sourcegems.tgz -C $gemdir\'

#Bwaila
#ssh nrb-admin@10.41.5.20 "cd /var/www/edrs_facility && gemdir=`gem env | grep "\\- INSTALLATION DIRECTORY" | awk -F \': \' {\'print $2\'}`"
#ssh nrb-admin@10.41.5.20 \'cd /var/www/edrs_facility && mkdir -p `gem env | grep "\\- INSTALLATION DIRECTORY" | awk -F \': \' {\'print $2\'}`\'
#ssh nrb-admin@10.41.5.20 \'cd /var/www/edrs_facility && tar xvfz sourcegems.tgz -C `gem env | grep "\\- INSTALLATION DIRECTORY" | awk -F \': \' {\'print $2\'}`\'
'''
      }
    }

    stage('Installing Ruby Gems') {
      steps {
        sh '''#Salima
#ssh nrb-admin@10.41.154.4 \'cd /var/www/edrs_facility && rm Gemfile.lock\'
#ssh nrb-admin@10.41.154.4 \'cd /var/www/edrs_facility && ./setup.sh\'
#python edrs_setup.py
'''
      }
    }

  }
}