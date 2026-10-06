Organization                    | Company Name
│                               |
├── User                        | User name for application
│                               |
├── Employee                    | Employee Information
│    └── Role                   | Employee Role within application (will grant permissions)
│                               |
├── Recipe                      | Recipe name/style/etc
│    └── Recipe Version         | Recipe ingredients/process as ver. to trackchanges over time                       
│         ├── Grain Bill        |                
│         ├── Hop Addition      |                
│         ├── Yeast             |        
│         └── Water Chemistry   |                    
│                               |
├── Batch                       |  Batch information
│    ├── Recipe Version         |            
│    ├── Brew                   |    
│    ├── Fermentation Events    |                    
│    ├── Dry Hop Events         |            
│    ├── Yeast Harvest          |            
│    ├── QC                     |
│    └── Packaging              |        
│                               |
└── Tank                        |  Tank usage and maintenance log
     └── Tank History           |            
          └── Maintenance       |                

### Database relationships

Organization → Employees            one to many
Organization → Users                one to many
Employee → Role                     one to one
Employee → Employee (supervisor)    one to many
Recipe → Recipe Versions            one to many
Recipe Version → Grain Bills        one to many
Recipe Version → Hop Additions      one to many
Recipe Version → Yeast              one to many
Recipe Version → Water Chemistry    one to many
Batch → Recipe Version              one to many
Batch → Employee (brewer)           one to many
Batch → Fermentation Events         one to many
Batch → Packaging                   one to many
Tank → Tank History                 one to many
Tank History → Batch                one to many