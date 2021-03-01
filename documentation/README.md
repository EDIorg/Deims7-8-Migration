# Sequence of manual migration steps:
Don't forget after any changes to .yml files in the install folder run:
```
drush cim -y --partial --source=modules/custom/deims_migrate/config/install/
```
All drush commands are run from the webroot of the 'newsite', e.g. for Ubuntu - /var/www/newsite
In this document drush migrate:import is used to do the imports.  A GUI is also available, /admin/structure/migrate which mirrors the drush cammands. However the csv .yml files do not show up in the GUI.
It is a good idea to do backups between imports.  

1. If desired migrate users.
	1. Both  migrate_plus.migration.upgrade_d7_user.yml and migrate_plus.migration.upgrade_d7_user_role.yml files should be in the install folder.
	1. Run the following command.  Users will be import along with their roles.
	
	`drush migrate:import upgrade_d7_user'

1. Migrate taxonomies
	1. At NTL and ARC we have a few specific ones.  Edit the .yml files or use the upgrade_d7 files for specific taxonomies.  If .yml files were genrated with drush cim you will need to edit the files to remove the first line with the uuid number and edit parent_id changing it to: 
	`plugin: default_value`  
	`default_value: taxonomy_name`
	
	Where taxonomy_name is the name of the taxonomy being migrated.  This fixes the few cases where the imports failed with the message: SQLSTATE[23000]: Integrity constraint violation: 1048 Column 'vid' cannot be null:  
	1. Moving taxonomies separately allows to omit a few that seemed unnecessary. 
	1. Create taxonomies in D8 site: core_areas, lter_controlled_vocabulary, other custom ones.  If all taxonomies will be migradred you can use the upgrade_d7_taxonomy_vocabulary.yml file to create the taxonomies. 
	1. On the commandline inside the webroot of the new D8 website run the command 
	
	`drush migrate:import deims_category_core_areas` change migration ID for the others.

1. Migrate file entity information
	1. This yml file will only migrate information in the table: file_managed. It will not actually copy the files.
    1. On the commandline inside the webroot of the new D8 website run the command 
    
    `drush migrate:import deims_files`
	
1. Migrate basic pages and other custom content types that only need taxonomy tagging (e.g., research highlights, protocols, etc.)
	1. Create the desired content type in D8. For ARC the D7 simple page content was migrated to basic page in D8. The D8 basic page content included the combined fields.
    	1. Navigate in your D8 website to /admin/structure/types
    	1. Add Content type
    	1. Add needed fields.  For example:
    		* label: File; machine name: field_page_file; field type: File
    		* label: NTL Keyword; machine name: field_ntl_keyword; field type: Entity reference
		* For ARC site - label: Project Keyword; machine name: field_page_project_keyword; field type: Entity reference
    1. On the commandline inside the webroot of the new D8 website run the command 
    
        `drush migrate:import deims_nodes_highlights` 
    
     * Change migration name for the others. Note for the ARC migration, body format of filtered html did not migrate to D8's restricted html. After trying various ways of mapping the format the I used phpmyadmin SQL to replace filtered html with restricted html.
    
        `UPDATE `node__body` SET `body_format` = REPLACE(`body_format`, 'filtered_html', 'restricted_html') WHERE `body_format` LIKE '%filtered_html%'`

1. Migrate organizations   
	1. Create content type in D8 name: Organization; machine name: organization
    	1. Navigate in your D8 website to /admin/structure/types
    	1. Add Content type
    	1. It doesn't need any fields, only the title
    1. On the commandline inside the webroot of the new D8 website run the command 
    
    `drush migrate:import deims_nodes_organization`.


1. Migrate Person  
	1. Create content type in D8  
    		* name: Person; machine name: person
    	1. NTL: fields 
    		* label: Administrative Area; machine name: field_address_admin_area; type: Text (plain)
    		* label: City; machine name: field_address_locality; type: Text (plain)
    		* label: Country; machine name: field_address_country; type: Text (plain)
    		* label: Department; machine name: field_person_department; type: Text (plain)
    		* label: e-mail; machine name: field_person_email; type: Email
    		* label: Family Name; machine name: field_name_family; type: Text (plain)
    		* label: Given Name; machine name: field_name_given; type: Text (plain)
    		* label: List in directory; machine name: field_person_list_in_directory; type: Boolean
    		* label: Middle Name; machine name: field_name_middle; type: Text (plain)
    		* label: ORCID; machine name: field_person_orcid; type: Link
    		* label: Organization; machine name: field_organization; type: Entity reference
    		* label: Phone; machine name: field_person_phone; type: Telephone number
    		* label: Postal Code; machine name: field_address_postal_code; type: Text (plain)
    		* label: Premise; machine name: field_address_premise; type: Text (plain)
    		* label: Project Role; machine name: field_person_project_role; type: List (text)
    			* add list items: e.g. 'LPI|Lead Principal Investigator', 'COPI|co-Principal Investigator',
    				'FA|Faculty Associate', 'PDA|Post Doctoral Associate', 'OP|Other Professional',
    				'GS|Graduate Student', 'US|Undergraduate Student', 'OS|Other Staff', 'SC|Secretary Clerical',
    				'DA|Data Manager', 'PUB|Publisher', 'CO|Contact Person'
    		* label: Specialty; machine name: field_person_specialty; type: Text (plain)
    		* label: Street Address; machine name: field_address_street; type: Text (plain)		
    	1. ARC: The address module is used which Provides functionality for storing, validating and displaying international postal addresses. (composer require 'drupal/address:^1.8') The fields are
    		* label: Address; machine name: field_person_address; type: Address
    		* label: e-mail; machine name: field_person_email; type: Email
    		* label: List in directory; machine name: field_person_list_in_directory; type: Boolean
    		* label: ORCID; machine name: field_person_orcid; type: Link
    		* label: Organization; machine name: field_organization; type: Entity reference
    		* label: Phone; machine name: field_person_phone; type: Telephone number
    		* label: Project Role; machine name: field_person_project_role; type: List (text)
    			* add list items: e.g. 'LPI|Lead Principal Investigator', 'COPI|co-Principal Investigator',
    				'FA|Faculty Associate', 'PDA|Post Doctoral Associate', 'OP|Other Professional',
    				'GS|Graduate Student', 'US|Undergraduate Student', 'OS|Other Staff', 'SC|Secretary Clerical',
    				'DA|Data Manager', 'PUB|Publisher', 'CO|Contact Person'
    		* label: URL; machine name: field_person_url; type: Link
	1. Export person information from DEIMS7 database with [personExport.sql](https://github.com/lter/Deims7-8-Migration/blob/master/SQLexport_queries/personExport.sql) and save as personExport.csv. ARC used a view to export the csv person file.  Excel is used to cleanup some inconsistences and older content. See the .yml file for additional notes.
    	1. In the migration YML file make sure the path to that csv file is set correctly
		1. On the commandline inside the webroot of the new D8 website run 
    
        `drush cim -y --partial --source=modules/custom/deims_migrate/config/install/`
    
        `drush migrate:import deims_csv_person`  or the id name of the .yml file

1. Migrate research site

   1. Navigate in your D8 website to /admin/structure/types

      1. Add Content type 
         - name: Research site; machine name: research_site

      **NTL**

      1. needed fields

         * label: Body; machine name: body; type: Text (formatted, long, with summary)
         * label: Bottom Latitude; machine name: field_coord_bottom_latitude; type: Number (float)
         * label: Elevation; machine name: field_elevation; type: Number (integer)
         * label: Left Longitude; machine name: field_coord_left_longitude; type: Number (float)
         * label: Rigth Longitude; machine name: field_coord_rigth_longitude; type: Number (float)
         * label: Top Latitude; machine name: field_coord_top_latitude; type: Number (float)

       2. Export research site information from DEIMS7 database with [research_siteExport.sql](https://github.com/lter/Deims7-8-Migration/blob/master/SQLexport_queries/research_siteExport.sql) and save as research_siteExport.csv

       3. In the migration YML file make sure the path to that csv file is set correctly On the commandline inside the webroot of the new D8 website run 

          `drush cim -y --partial --source=modules/custom/deims_migrate/config/install/`

          `drush migrate:import deims_csv_site`  or whatever the id name of the .yml file

      **ARC**

      1. needed fields using latest version of Geofield with a geofield d7d8 plugin module. The D7 site geofield needs to be version 2.4.

         * label: Body; machine name: body; type: Text (formatted, long, with summary)
         * label: Coordinates; machine name: field_coordinates; type: Geofield
         * label: Elevation; machine name: field_elevation; type: Number (integer)
         * label: Research Site Images; machine name: field_research_site_images; type: Image

      2. Use the upgrade_d7node_reserach_site.yml  as a template. Run

         `drush cim -y --partial --source=modules/custom/deims_migrate/config/install/`

         `drush migrate:import upgeade_d7node_reserach_site`  or whatever the id name of the .yml file

1. Migrate Project and create new content type for funding in anticipating of EML 2.2  
	1. Navigate in your D8 website to /admin/structure/types. Add Content type Project Funding  
		* label: Project Funding; machine name: project_funding
	2. Add needed fields  
		* label: Award URL; machine name: field_funder_award_url; type: Link
		* label: Funder Award Number; machine name: field_funder_award_number; type: Text (plain)
		* label: Funder Award Title; machine name: field_funder_award_title; type: Text (plain)
		* label: Funder ID; machine name: field_funder_id; type: Text (plain)
		* label: Funder Name; machine name: field_funder_name; type: Text (plain)

1. Add Content type Research Project  
	1.  Add content Research Project
		* label: Research Project; machine name: project
    	2. Add needed fields
	- **NTL**
		* label: Body; machine name: body Text (formatted, long, with summary)
		* label: Funding; machine name: field_project_funding; type: Entity reference
		* label: Investigator; machine name: field_project_investigator; type: Entity reference
		* label: LTER Keyword; machine name: field_project_lter_keyword; type: Entity reference
		* label: NTL Keyword; machine name: field_project_ntl_keyword; type: Entity reference
		* label: Timeline; machine name: field_project_timeline; type: Date range
	- **ARC**
		* label: Body; machine name: body; type: Text (formatted, long, with summary)
		* label: Funding; machine name: field_project_funding; type: Entity reference
		* label: Investigator; machine name: field_project_investigator; type: Entity reference
		* label: LTER Keyword; machine name: field_project_lter_keyword; type: Entity reference
		* label: ARC Keyword; machine name: field_project_arc_keyword; type: Entity reference
		* label: Timeline; machine name: field_project_timeline; type: Date range
		* label: Photos; machine name: field_project_photos; type: Image
		* label: Abstract; machine name :field_project_abstract; type: Text (formatted, long)
		* label: Project or Theme Keyword; machine name: field_project_or_theme_keyword; type: Entity reference
    2. On the commandline inside the webroot of the new D8 website run 

         `drush migrate:import deims_nodes_project`   or whatever the id name of the .yml file is.
	 
	 * Currently the YML script relies on the fact that old DIEMS7 nids are being used in D8 and no migration_lookup is performed!
	 * No funding information is in DEIMS7. ARC has funding numbers in D7 projects. The numbers were exported and excel used to add Award title and funder name.
    	
1. Migrate variable - this requires new content types and some R scripts. Detailed instructions are: [parseVariables](https://github.com/lter/Deims7-8-Migration/tree/master/documentation/parseVariables)

1. Migrate data sources
	1. Create content type in D8 name: Data Source machine name: data_source
    	1. Navigate in your D8 website to /admin/structure/types
    	1. Add Content type
    	1. Add needed fields 
    		* label: Database; machine name: field_dsource_de_database;type: Text (plain)
    		* lable: database table; machine name: field_dsource_de_table; type: Text (plain)
    		* label: Date Range; machine name: field_dsource_date_range; type: Date range 	
    		* label: Description; machine name: field_dsource_description; type: Text (plain, long) 	
    		* label: Field Delimiter; machine name: field_dsource_field_delimiter; type: List (text)
    			* add list items: ',|Comma (,)', '\t|tab', ';|Semicolon (;)', 'other|other'
    			* set the default value
    		* label: File Upload; machine name: field_dsource_file; type: File 	
    		* label: Footer Lines; machine name: field_dsource_footer_lines; type: Number (integer) 	
    		* label: Header Lines; machine name: field_dsource_header_lines; type: Number (integer) 	
    		* label: Instrumentation; machine name: field_dsource_instrumentation; type: Text (plain, long) 	
    		* label: Methods; machine name: field_dsource_methods; type: Text (plain, long) 	
    		* label: Number of Records; machine name: field_dsource_num_records; type: Number (integer) 	
    		* label: Orientation; machine name: field_dsource_orientation; type: List (text)
    			* add list items: 'column|column', 'row|row'
    			* set the default value
    		* label: Quality Assurance; machine name: field_dsource_quality_assurance; type: Text (plain, long) 	
    		* label: Quote Character; machine name: field_dsource_quote_character; type: List (text)
    			* add list items: ''|single quote (')','"|double quote (")', 'other|other'
    			* set the default value
    		* label: Record Delimiter; machine name: field_dsource_record_delimiter; type: List (text)
    			* add list items: '\n|Newline (Unix \n)', '\r\n|Newline (Windows \r\n)', '\r|Newline (some macs \r)', ';|Semicolon (;)', 'other|other'
    			* set the default value
    		* label: Related Sites; machine name: field_dsource_related_sites; type: Entity reference 	
    		* label: Variables; machine name: field_dsource_variables; type: Entity reference
    1. On the commandline inside the webroot of the new D8 website run `drush migrate:import deims_nodes_dsource`
    	* Currently the YML script relies on the fact that old DIEMS7 nids are being used in D8 and no migration_lookup is performed!
    1. Use [SQL script](https://github.com/lter/Deims7-8-Migration/blob/master/SQLexport_queries/exportDataSourceIDs.sql) on the D8 site to get the new nid/vid mapping. Save it as 'exportDataSourceIDs.csv' with headers.
    1. See last point in [parseVariables](https://github.com/lter/Deims7-8-Migration/tree/master/documentation/parseVariables). Run [R script](https://github.com/lter/Deims7-8-Migration/blob/master/R%20scripts/datasourceVariablesReference.R) to create the upload file needed to link variables to each data source.
    1. Manually upload file 'upload_dsourceVariablesReference.csv' to table: 'node__field_dsource_variables', e.g. using sql LOAD DATA for a file that is comma delimited, column in quotes and terminated by windows newline. 
    ```
    LOAD DATA INFILE 'upload_dsourceVariablesReference.csv' INTO TABLE `node__field_dsource_variables` 
    FIELDS TERMINATED BY ',' ENCLOSED BY '\"' LINES TERMINATED BY '\r\n' IGNORE 1 LINES 
    (`bundle`, `entity_id`, `revision_id`, `langcode`, `delta`, `field_dsource_variables_target_id`)
    ```    
    1. Clear all caches in the D8 site and make sure the data sources look like they are supposed to:
    	* They have all variables linked
    	* They have the csv file linked
1. Migrate data sets - [see documentation](https://github.com/lter/Deims7-8-Migration/tree/master/documentation/dataSet)   
