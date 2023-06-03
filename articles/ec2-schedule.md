---
title: "Instance Schedulerを使ってEC2の稼働時間を管理してみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","ec2"]
published: true
---

# 概要
会社で使っているEC2はSavings Plansを採用していてオンデマンドで使うよりも安い料金でEC2を利用しています。
ただそれでもEC2の数が段々と増えていて、当初の予想よりもコンピューティングリソースの消費が大きくなりEC2の利用料が大きくなってきました。
そこで検証環境のEC2の稼働時間を減らし、コンピューティングリソースの消費を抑えることでEC2の利用料を節約しようと思いました。
前職では先輩が作成したLambdaで21:00~翌9:00の時間帯は自動でシャットダウンする仕組みができており、当初はそれを真似しようと思いましたがInstance Schedulerなるものを今の会社のリーダーから教わりました。
今回はInstance Schedulerを実際に使ってみてどういったものなのか試してみました。

# Instance Schedulerについて
先にお話しするとInstance SchedulerというAWSリソースはありません。
複数のサービスを組み合わせて実装したAWSソリューションです。
https://aws.amazon.com/jp/solutions/implementations/instance-scheduler/

イメージとしてはEC2に専用のタグを付与して、LambdaがDynamoDBに保存されているタグごとに定義されたスケジュール情報を読み取り、DynamoDBのタグ情報と一致したタグを持つEC2に対してLambdaが停止、起動するという仕組みになっています。
![](/images/ec2-schedule/image1.png)
*アーキテクチャ*

[上記ページから引用](https://aws.amazon.com/jp/solutions/implementations/instance-scheduler/)
CloudFormationテンプレートファイルが用意されていますので、自前で上記アーキテクチャを構築する必要はなく、簡単に作成できますのでまずはテンプレートファイルをダウンロードし、CloudFormationを実行します。

:::details テンプレートファイル
```json:
{
 "Description": "(SO0030) - The AWS CloudFormation template for deployment of the aws-instance-scheduler, version: 1.5.0",
 "AWSTemplateFormatVersion": "2010-09-09",
 "Metadata": {
  "AWS::CloudFormation::Interface": {
   "ParameterGroups": [
    {
     "Label": {
      "default": "Scheduler (version 1.5.0)"
     },
     "Parameters": [
      "TagName",
      "ScheduledServices",
      "ScheduleRdsClusters",
      "CreateRdsSnapshot",
      "SchedulingActive",
      "DefaultTimezone",
      "CrossAccountRoles",
      "ScheduleLambdaAccount",
      "SchedulerFrequency",
      "MemorySize"
     ]
    },
    {
     "Label": {
      "default": "Namespace Configuration"
     },
     "Parameters": [
      "Namespace"
     ]
    },
    {
     "Label": {
      "default": "Account Structure"
     },
     "Parameters": [
      "UsingAWSOrganizations",
      "Principals",
      "Regions"
     ]
    },
    {
     "Label": {
      "default": "Options"
     },
     "Parameters": [
      "UseCloudWatchMetrics",
      "Trace",
      "EnableSSMMaintenanceWindows"
     ]
    },
    {
     "Label": {
      "default": "Other parameters"
     },
     "Parameters": [
      "LogRetentionDays",
      "StartedTags",
      "StoppedTags"
     ]
    }
   ],
   "ParameterLabels": {
    "Namespace": {
     "default": "Provide a unique namespace value."
    },
    "LogRetentionDays": {
     "default": "Log retention days"
    },
    "StartedTags": {
     "default": "Started tags"
    },
    "StoppedTags": {
     "default": "Stopped tags"
    },
    "SchedulingActive": {
     "default": "Scheduling enabled"
    },
    "UsingAWSOrganizations": {
     "default": "Using AWS Organizations?"
    },
    "Principals": {
     "default": "Provide Organization Id OR List of Remote Account Ids"
    },
    "ScheduleLambdaAccount": {
     "default": "This account"
    },
    "UseCloudWatchMetrics": {
     "default": "Enable CloudWatch Metrics"
    },
    "Trace": {
     "default": "Enable CloudWatch Debug Logs"
    },
    "EnableSSMMaintenanceWindows": {
     "default": "Enable SSM Maintenance windows"
    },
    "TagName": {
     "default": "Instance Scheduler tag name"
    },
    "ScheduledServices": {
     "default": "Service(s) to schedule"
    },
    "ScheduleRdsClusters": {
     "default": "Schedule Aurora Clusters"
    },
    "CreateRdsSnapshot": {
     "default": "Create RDS instance snapshot"
    },
    "DefaultTimezone": {
     "default": "Default time zone"
    },
    "SchedulerFrequency": {
     "default": "Frequency"
    },
    "Regions": {
     "default": "Region(s)"
    },
    "MemorySize": {
     "default": "Memory size"
    }
   }
  }
 },
 "Parameters": {
  "SchedulingActive": {
   "Type": "String",
   "Default": "Yes",
   "AllowedValues": [
    "Yes",
    "No"
   ],
   "Description": "Activate or deactivate scheduling."
  },
  "ScheduledServices": {
   "Type": "String",
   "Default": "EC2",
   "AllowedValues": [
    "EC2",
    "RDS",
    "Both"
   ],
   "Description": "Scheduled Services."
  },
  "ScheduleRdsClusters": {
   "Type": "String",
   "Default": "No",
   "AllowedValues": [
    "Yes",
    "No"
   ],
   "Description": "Enable scheduling of Aurora clusters for RDS Service."
  },
  "CreateRdsSnapshot": {
   "Type": "String",
   "Default": "No",
   "AllowedValues": [
    "Yes",
    "No"
   ],
   "Description": "Create snapshot before stopping RDS instances (does not apply to Aurora Clusters)."
  },
  "MemorySize": {
   "Type": "Number",
   "Default": 128,
   "AllowedValues": [
    "128",
    "384",
    "512",
    "640",
    "768",
    "896",
    "1024",
    "1152",
    "1280",
    "1408",
    "1536"
   ],
   "Description": "Size of the Lambda function running the scheduler, increase size when processing large numbers of instances."
  },
  "UseCloudWatchMetrics": {
   "Type": "String",
   "Default": "No",
   "AllowedValues": [
    "Yes",
    "No"
   ],
   "Description": "Collect instance scheduling data using CloudWatch metrics."
  },
  "LogRetentionDays": {
   "Type": "Number",
   "Default": 30,
   "AllowedValues": [
    "1",
    "3",
    "5",
    "7",
    "14",
    "30",
    "60",
    "90",
    "120",
    "150",
    "180",
    "365",
    "400",
    "545",
    "731",
    "1827",
    "3653"
   ],
   "Description": "Retention days for scheduler logs."
  },
  "Trace": {
   "Type": "String",
   "Default": "No",
   "AllowedValues": [
    "Yes",
    "No"
   ],
   "Description": "Enable debug-level logging in CloudWatch logs."
  },
  "EnableSSMMaintenanceWindows": {
   "Type": "String",
   "Default": "No",
   "AllowedValues": [
    "Yes",
    "No"
   ],
   "Description": "Enable the solution to load SSM Maintenance Windows, so that they can be used for EC2 instance Scheduling."
  },
  "TagName": {
   "Type": "String",
   "Default": "Schedule",
   "Description": "Name of tag to use for associating instance schedule schemas with service instances.",
   "MaxLength": 127,
   "MinLength": 1
  },
  "DefaultTimezone": {
   "Type": "String",
   "Default": "UTC",
   "AllowedValues": [
    "Africa/Abidjan",
    "Africa/Accra",
    "Africa/Addis_Ababa",
    "Africa/Algiers",
    "Africa/Asmara",
    "Africa/Bamako",
    "Africa/Bangui",
    "Africa/Banjul",
    "Africa/Bissau",
    "Africa/Blantyre",
    "Africa/Brazzaville",
    "Africa/Bujumbura",
    "Africa/Cairo",
    "Africa/Casablanca",
    "Africa/Ceuta",
    "Africa/Conakry",
    "Africa/Dakar",
    "Africa/Dar_es_Salaam",
    "Africa/Djibouti",
    "Africa/Douala",
    "Africa/El_Aaiun",
    "Africa/Freetown",
    "Africa/Gaborone",
    "Africa/Harare",
    "Africa/Johannesburg",
    "Africa/Juba",
    "Africa/Kampala",
    "Africa/Khartoum",
    "Africa/Kigali",
    "Africa/Kinshasa",
    "Africa/Lagos",
    "Africa/Libreville",
    "Africa/Lome",
    "Africa/Luanda",
    "Africa/Lubumbashi",
    "Africa/Lusaka",
    "Africa/Malabo",
    "Africa/Maputo",
    "Africa/Maseru",
    "Africa/Mbabane",
    "Africa/Mogadishu",
    "Africa/Monrovia",
    "Africa/Nairobi",
    "Africa/Ndjamena",
    "Africa/Niamey",
    "Africa/Nouakchott",
    "Africa/Ouagadougou",
    "Africa/Porto-Novo",
    "Africa/Sao_Tome",
    "Africa/Tripoli",
    "Africa/Tunis",
    "Africa/Windhoek",
    "America/Adak",
    "America/Anchorage",
    "America/Anguilla",
    "America/Antigua",
    "America/Araguaina",
    "America/Argentina/Buenos_Aires",
    "America/Argentina/Catamarca",
    "America/Argentina/Cordoba",
    "America/Argentina/Jujuy",
    "America/Argentina/La_Rioja",
    "America/Argentina/Mendoza",
    "America/Argentina/Rio_Gallegos",
    "America/Argentina/Salta",
    "America/Argentina/San_Juan",
    "America/Argentina/San_Luis",
    "America/Argentina/Tucuman",
    "America/Argentina/Ushuaia",
    "America/Aruba",
    "America/Asuncion",
    "America/Atikokan",
    "America/Bahia",
    "America/Bahia_Banderas",
    "America/Barbados",
    "America/Belem",
    "America/Belize",
    "America/Blanc-Sablon",
    "America/Boa_Vista",
    "America/Bogota",
    "America/Boise",
    "America/Cambridge_Bay",
    "America/Campo_Grande",
    "America/Cancun",
    "America/Caracas",
    "America/Cayenne",
    "America/Cayman",
    "America/Chicago",
    "America/Chihuahua",
    "America/Costa_Rica",
    "America/Creston",
    "America/Cuiaba",
    "America/Curacao",
    "America/Danmarkshavn",
    "America/Dawson",
    "America/Dawson_Creek",
    "America/Denver",
    "America/Detroit",
    "America/Dominica",
    "America/Edmonton",
    "America/Eirunepe",
    "America/El_Salvador",
    "America/Fortaleza",
    "America/Glace_Bay",
    "America/Godthab",
    "America/Goose_Bay",
    "America/Grand_Turk",
    "America/Grenada",
    "America/Guadeloupe",
    "America/Guatemala",
    "America/Guayaquil",
    "America/Guyana",
    "America/Halifax",
    "America/Havana",
    "America/Hermosillo",
    "America/Indiana/Indianapolis",
    "America/Indiana/Knox",
    "America/Indiana/Marengo",
    "America/Indiana/Petersburg",
    "America/Indiana/Tell_City",
    "America/Indiana/Vevay",
    "America/Indiana/Vincennes",
    "America/Indiana/Winamac",
    "America/Inuvik",
    "America/Iqaluit",
    "America/Jamaica",
    "America/Juneau",
    "America/Kentucky/Louisville",
    "America/Kentucky/Monticello",
    "America/Kralendijk",
    "America/La_Paz",
    "America/Lima",
    "America/Los_Angeles",
    "America/Lower_Princes",
    "America/Maceio",
    "America/Managua",
    "America/Manaus",
    "America/Marigot",
    "America/Martinique",
    "America/Matamoros",
    "America/Mazatlan",
    "America/Menominee",
    "America/Merida",
    "America/Metlakatla",
    "America/Mexico_City",
    "America/Miquelon",
    "America/Moncton",
    "America/Monterrey",
    "America/Montevideo",
    "America/Montreal",
    "America/Montserrat",
    "America/Nassau",
    "America/New_York",
    "America/Nipigon",
    "America/Nome",
    "America/Noronha",
    "America/North_Dakota/Beulah",
    "America/North_Dakota/Center",
    "America/North_Dakota/New_Salem",
    "America/Ojinaga",
    "America/Panama",
    "America/Pangnirtung",
    "America/Paramaribo",
    "America/Phoenix",
    "America/Port-au-Prince",
    "America/Port_of_Spain",
    "America/Porto_Velho",
    "America/Puerto_Rico",
    "America/Rainy_River",
    "America/Rankin_Inlet",
    "America/Recife",
    "America/Regina",
    "America/Resolute",
    "America/Rio_Branco",
    "America/Santa_Isabel",
    "America/Santarem",
    "America/Santiago",
    "America/Santo_Domingo",
    "America/Sao_Paulo",
    "America/Scoresbysund",
    "America/Sitka",
    "America/St_Barthelemy",
    "America/St_Johns",
    "America/St_Kitts",
    "America/St_Lucia",
    "America/St_Thomas",
    "America/St_Vincent",
    "America/Swift_Current",
    "America/Tegucigalpa",
    "America/Thule",
    "America/Thunder_Bay",
    "America/Tijuana",
    "America/Toronto",
    "America/Tortola",
    "America/Vancouver",
    "America/Whitehorse",
    "America/Winnipeg",
    "America/Yakutat",
    "America/Yellowknife",
    "Antarctica/Casey",
    "Antarctica/Davis",
    "Antarctica/DumontDUrville",
    "Antarctica/Macquarie",
    "Antarctica/Mawson",
    "Antarctica/McMurdo",
    "Antarctica/Palmer",
    "Antarctica/Rothera",
    "Antarctica/Syowa",
    "Antarctica/Vostok",
    "Arctic/Longyearbyen",
    "Asia/Aden",
    "Asia/Almaty",
    "Asia/Amman",
    "Asia/Anadyr",
    "Asia/Aqtau",
    "Asia/Aqtobe",
    "Asia/Ashgabat",
    "Asia/Baghdad",
    "Asia/Bahrain",
    "Asia/Baku",
    "Asia/Bangkok",
    "Asia/Beirut",
    "Asia/Bishkek",
    "Asia/Brunei",
    "Asia/Choibalsan",
    "Asia/Chongqing",
    "Asia/Colombo",
    "Asia/Damascus",
    "Asia/Dhaka",
    "Asia/Dili",
    "Asia/Dubai",
    "Asia/Dushanbe",
    "Asia/Gaza",
    "Asia/Harbin",
    "Asia/Hebron",
    "Asia/Ho_Chi_Minh",
    "Asia/Hong_Kong",
    "Asia/Hovd",
    "Asia/Irkutsk",
    "Asia/Jakarta",
    "Asia/Jayapura",
    "Asia/Jerusalem",
    "Asia/Kabul",
    "Asia/Kamchatka",
    "Asia/Karachi",
    "Asia/Kashgar",
    "Asia/Kathmandu",
    "Asia/Khandyga",
    "Asia/Kolkata",
    "Asia/Krasnoyarsk",
    "Asia/Kuala_Lumpur",
    "Asia/Kuching",
    "Asia/Kuwait",
    "Asia/Macau",
    "Asia/Magadan",
    "Asia/Makassar",
    "Asia/Manila",
    "Asia/Muscat",
    "Asia/Nicosia",
    "Asia/Novokuznetsk",
    "Asia/Novosibirsk",
    "Asia/Omsk",
    "Asia/Oral",
    "Asia/Phnom_Penh",
    "Asia/Pontianak",
    "Asia/Pyongyang",
    "Asia/Qatar",
    "Asia/Qyzylorda",
    "Asia/Rangoon",
    "Asia/Riyadh",
    "Asia/Sakhalin",
    "Asia/Samarkand",
    "Asia/Seoul",
    "Asia/Shanghai",
    "Asia/Singapore",
    "Asia/Taipei",
    "Asia/Tashkent",
    "Asia/Tbilisi",
    "Asia/Tehran",
    "Asia/Thimphu",
    "Asia/Tokyo",
    "Asia/Ulaanbaatar",
    "Asia/Urumqi",
    "Asia/Ust-Nera",
    "Asia/Vientiane",
    "Asia/Vladivostok",
    "Asia/Yakutsk",
    "Asia/Yekaterinburg",
    "Asia/Yerevan",
    "Atlantic/Azores",
    "Atlantic/Bermuda",
    "Atlantic/Canary",
    "Atlantic/Cape_Verde",
    "Atlantic/Faroe",
    "Atlantic/Madeira",
    "Atlantic/Reykjavik",
    "Atlantic/South_Georgia",
    "Atlantic/St_Helena",
    "Atlantic/Stanley",
    "Australia/Adelaide",
    "Australia/Brisbane",
    "Australia/Broken_Hill",
    "Australia/Currie",
    "Australia/Darwin",
    "Australia/Eucla",
    "Australia/Hobart",
    "Australia/Lindeman",
    "Australia/Lord_Howe",
    "Australia/Melbourne",
    "Australia/Perth",
    "Australia/Sydney",
    "Canada/Atlantic",
    "Canada/Central",
    "Canada/Eastern",
    "Canada/Mountain",
    "Canada/Newfoundland",
    "Canada/Pacific",
    "Europe/Amsterdam",
    "Europe/Andorra",
    "Europe/Athens",
    "Europe/Belgrade",
    "Europe/Berlin",
    "Europe/Bratislava",
    "Europe/Brussels",
    "Europe/Bucharest",
    "Europe/Budapest",
    "Europe/Busingen",
    "Europe/Chisinau",
    "Europe/Copenhagen",
    "Europe/Dublin",
    "Europe/Gibraltar",
    "Europe/Guernsey",
    "Europe/Helsinki",
    "Europe/Isle_of_Man",
    "Europe/Istanbul",
    "Europe/Jersey",
    "Europe/Kaliningrad",
    "Europe/Kiev",
    "Europe/Lisbon",
    "Europe/Ljubljana",
    "Europe/London",
    "Europe/Luxembourg",
    "Europe/Madrid",
    "Europe/Malta",
    "Europe/Mariehamn",
    "Europe/Minsk",
    "Europe/Monaco",
    "Europe/Moscow",
    "Europe/Oslo",
    "Europe/Paris",
    "Europe/Podgorica",
    "Europe/Prague",
    "Europe/Riga",
    "Europe/Rome",
    "Europe/Samara",
    "Europe/San_Marino",
    "Europe/Sarajevo",
    "Europe/Simferopol",
    "Europe/Skopje",
    "Europe/Sofia",
    "Europe/Stockholm",
    "Europe/Tallinn",
    "Europe/Tirane",
    "Europe/Uzhgorod",
    "Europe/Vaduz",
    "Europe/Vatican",
    "Europe/Vienna",
    "Europe/Vilnius",
    "Europe/Volgograd",
    "Europe/Warsaw",
    "Europe/Zagreb",
    "Europe/Zaporozhye",
    "Europe/Zurich",
    "GMT",
    "Indian/Antananarivo",
    "Indian/Chagos",
    "Indian/Christmas",
    "Indian/Cocos",
    "Indian/Comoro",
    "Indian/Kerguelen",
    "Indian/Mahe",
    "Indian/Maldives",
    "Indian/Mauritius",
    "Indian/Mayotte",
    "Indian/Reunion",
    "Pacific/Apia",
    "Pacific/Auckland",
    "Pacific/Chatham",
    "Pacific/Chuuk",
    "Pacific/Easter",
    "Pacific/Efate",
    "Pacific/Enderbury",
    "Pacific/Fakaofo",
    "Pacific/Fiji",
    "Pacific/Funafuti",
    "Pacific/Galapagos",
    "Pacific/Gambier",
    "Pacific/Guadalcanal",
    "Pacific/Guam",
    "Pacific/Honolulu",
    "Pacific/Johnston",
    "Pacific/Kiritimati",
    "Pacific/Kosrae",
    "Pacific/Kwajalein",
    "Pacific/Majuro",
    "Pacific/Marquesas",
    "Pacific/Midway",
    "Pacific/Nauru",
    "Pacific/Niue",
    "Pacific/Norfolk",
    "Pacific/Noumea",
    "Pacific/Pago_Pago",
    "Pacific/Palau",
    "Pacific/Pitcairn",
    "Pacific/Pohnpei",
    "Pacific/Port_Moresby",
    "Pacific/Rarotonga",
    "Pacific/Saipan",
    "Pacific/Tahiti",
    "Pacific/Tarawa",
    "Pacific/Tongatapu",
    "Pacific/Wake",
    "Pacific/Wallis",
    "US/Alaska",
    "US/Arizona",
    "US/Central",
    "US/Eastern",
    "US/Hawaii",
    "US/Mountain",
    "US/Pacific",
    "UTC"
   ],
   "Description": "Choose the default Time Zone. Default is 'UTC'."
  },
  "Regions": {
   "Type": "CommaDelimitedList",
   "Default": "",
   "Description": "List of regions in which instances are scheduled, leave blank for current region only."
  },
  "UsingAWSOrganizations": {
   "Type": "String",
   "Default": "No",
   "AllowedValues": [
    "Yes",
    "No"
   ],
   "Description": "Is the hub account member of the AWS Organizations?"
  },
  "Principals": {
   "Type": "CommaDelimitedList",
   "Default": "",
   "Description": "(Required) If using AWS Organizations, provide Organization ID. Eg. o-xxxxyyy. Else, comma separated list of remote account ids. Eg.: 1111111111, 2222222222 or {param: ssm-param-name}"
  },
  "Namespace": {
   "Type": "String",
   "Default": "",
   "Description": "Provide unique identifier to differentiate between multiple solution deployments (No Spaces). Example: Dev"
  },
  "StartedTags": {
   "Type": "String",
   "Default": "",
   "Description": "Comma separated list of tagname and values on the formt name=value,name=value,.. that are set on started instances"
  },
  "StoppedTags": {
   "Type": "String",
   "Default": "",
   "Description": "Comma separated list of tagname and values on the formt name=value,name=value,.. that are set on stopped instances"
  },
  "SchedulerFrequency": {
   "Type": "String",
   "Default": "5",
   "AllowedValues": [
    "1",
    "2",
    "5",
    "10",
    "15",
    "30",
    "60"
   ],
   "Description": "Scheduler running frequency in minutes."
  },
  "ScheduleLambdaAccount": {
   "Type": "String",
   "Default": "Yes",
   "AllowedValues": [
    "Yes",
    "No"
   ],
   "Description": "Schedule instances in this account."
  }
 },
 "Conditions": {
  "IsMemberOfOrganization": {
   "Fn::Equals": [
    {
     "Ref": "UsingAWSOrganizations"
    },
    "Yes"
   ]
  }
 },
 "Mappings": {
  "mappings": {
   "TrueFalse": {
    "Yes": "True",
    "No": "False"
   },
   "EnabledDisabled": {
    "Yes": "ENABLED",
    "No": "DISABLED"
   },
   "Services": {
    "EC2": "ec2",
    "RDS": "rds",
    "Both": "ec2,rds"
   },
   "Timeouts": {
    "1": "cron(0/1 * * * ? *)",
    "2": "cron(0/2 * * * ? *)",
    "5": "cron(0/5 * * * ? *)",
    "10": "cron(0/10 * * * ? *)",
    "15": "cron(0/15 * * * ? *)",
    "30": "cron(0/30 * * * ? *)",
    "60": "cron(0 0/1 * * ? *)"
   },
   "Settings": {
    "MetricsUrl": "https://metrics.awssolutionsbuilder.com/generic",
    "MetricsSolutionId": "S00030"
   },
   "SchedulerRole": {
    "Name": "Scheduler-Role"
   },
   "SchedulerEventBusName": {
    "Name": "scheduler-event-bus"
   }
  },
  "Send": {
   "AnonymousUsage": {
    "Data": "Yes"
   },
   "ParameterKey": {
    "UniqueId": "/Solutions/aws-instance-scheduler/UUID/"
   }
  },
  "AppRegistryForInstanceSchedulerSolution25A90F05": {
   "Data": {
    "ID": "SO0030",
    "Version": "1.5.0",
    "AppRegistryApplicationName": "instance-scheduler-on-aws",
    "SolutionName": "aws-instance-scheduler",
    "ApplicationType": "AWS-Solutions"
   }
  }
 },
 "Resources": {
  "AppRegistryForInstanceSchedulerDefaultApplicationAttributes78274898": {
   "Type": "AWS::ServiceCatalogAppRegistry::AttributeGroup",
   "Properties": {
    "Attributes": {
     "applicationType": {
      "Fn::FindInMap": [
       "AppRegistryForInstanceSchedulerSolution25A90F05",
       "Data",
       "ApplicationType"
      ]
     },
     "version": {
      "Fn::FindInMap": [
       "AppRegistryForInstanceSchedulerSolution25A90F05",
       "Data",
       "Version"
      ]
     },
     "solutionID": {
      "Fn::FindInMap": [
       "AppRegistryForInstanceSchedulerSolution25A90F05",
       "Data",
       "ID"
      ]
     },
     "solutionName": {
      "Fn::FindInMap": [
       "AppRegistryForInstanceSchedulerSolution25A90F05",
       "Data",
       "SolutionName"
      ]
     }
    },
    "Name": {
     "Fn::Join": [
      "-",
      [
       {
        "Ref": "AWS::Region"
       },
       {
        "Ref": "AWS::StackName"
       }
      ]
     ]
    },
    "Description": "Attribute group for solution information"
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/AppRegistryForInstanceScheduler/DefaultApplicationAttributes/Resource"
   }
  },
  "AppRegistry968496A3": {
   "Type": "AWS::ServiceCatalogAppRegistry::Application",
   "Properties": {
    "Name": {
     "Fn::Join": [
      "-",
      [
       {
        "Fn::FindInMap": [
         "AppRegistryForInstanceSchedulerSolution25A90F05",
         "Data",
         "AppRegistryApplicationName"
        ]
       },
       {
        "Ref": "AWS::Region"
       },
       {
        "Ref": "AWS::AccountId"
       },
       {
        "Ref": "AWS::StackName"
       }
      ]
     ]
    },
    "Description": {
     "Fn::Join": [
      "",
      [
       "Service Catalog application to track and manage all your resources for the solution ",
       {
        "Fn::FindInMap": [
         "AppRegistryForInstanceSchedulerSolution25A90F05",
         "Data",
         "SolutionName"
        ]
       }
      ]
     ]
    },
    "Tags": {
     "Solutions:ApplicationType": {
      "Fn::FindInMap": [
       "AppRegistryForInstanceSchedulerSolution25A90F05",
       "Data",
       "ApplicationType"
      ]
     },
     "Solutions:SolutionID": {
      "Fn::FindInMap": [
       "AppRegistryForInstanceSchedulerSolution25A90F05",
       "Data",
       "ID"
      ]
     },
     "Solutions:SolutionName": {
      "Fn::FindInMap": [
       "AppRegistryForInstanceSchedulerSolution25A90F05",
       "Data",
       "SolutionName"
      ]
     },
     "Solutions:SolutionVersion": {
      "Fn::FindInMap": [
       "AppRegistryForInstanceSchedulerSolution25A90F05",
       "Data",
       "Version"
      ]
     }
    }
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/AppRegistry/Resource"
   }
  },
  "AppRegistryResourceAssociationaf0b8dafb9748CD626E9": {
   "Type": "AWS::ServiceCatalogAppRegistry::ResourceAssociation",
   "Properties": {
    "Application": {
     "Fn::GetAtt": [
      "AppRegistry968496A3",
      "Id"
     ]
    },
    "Resource": {
     "Ref": "AWS::StackId"
    },
    "ResourceType": "CFN_STACK"
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/AppRegistry/ResourceAssociationaf0b8dafb974"
   }
  },
  "AppRegistryAttributeGroupAssociation9b87a9e8202fDED525C6": {
   "Type": "AWS::ServiceCatalogAppRegistry::AttributeGroupAssociation",
   "Properties": {
    "Application": {
     "Fn::GetAtt": [
      "AppRegistry968496A3",
      "Id"
     ]
    },
    "AttributeGroup": {
     "Fn::GetAtt": [
      "AppRegistryForInstanceSchedulerDefaultApplicationAttributes78274898",
      "Id"
     ]
    }
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/AppRegistry/AttributeGroupAssociation9b87a9e8202f"
   }
  },
  "SchedulerLogGroup": {
   "Type": "AWS::Logs::LogGroup",
   "Properties": {
    "LogGroupName": {
     "Fn::Join": [
      "",
      [
       {
        "Ref": "AWS::StackName"
       },
       "-logs"
      ]
     ]
    },
    "RetentionInDays": {
     "Ref": "LogRetentionDays"
    }
   },
   "UpdateReplacePolicy": "Delete",
   "DeletionPolicy": "Delete",
   "Metadata": {
    "cfn_nag": {
     "rules_to_suppress": [
      {
       "id": "W84",
       "reason": "CloudWatch log groups only have transactional data from the Lambda function, this template has to be supported in gov cloud which doesn't yet have the feature to provide kms key id to cloudwatch log group."
      }
     ]
    }
   }
  },
  "SchedulerRole": {
   "Type": "AWS::IAM::Role",
   "Properties": {
    "AssumeRolePolicyDocument": {
     "Statement": [
      {
       "Action": "sts:AssumeRole",
       "Effect": "Allow",
       "Principal": {
        "Service": "events.amazonaws.com"
       }
      },
      {
       "Action": "sts:AssumeRole",
       "Effect": "Allow",
       "Principal": {
        "Service": "lambda.amazonaws.com"
       }
      }
     ],
     "Version": "2012-10-17"
    },
    "Path": "/"
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/SchedulerRole/Resource"
   }
  },
  "SchedulerRoleDefaultPolicy66F774B8": {
   "Type": "AWS::IAM::Policy",
   "Properties": {
    "PolicyDocument": {
     "Statement": [
      {
       "Action": [
        "xray:PutTraceSegments",
        "xray:PutTelemetryRecords"
       ],
       "Effect": "Allow",
       "Resource": "*"
      },
      {
       "Action": [
        "dynamodb:BatchGetItem",
        "dynamodb:GetRecords",
        "dynamodb:GetShardIterator",
        "dynamodb:Query",
        "dynamodb:GetItem",
        "dynamodb:Scan",
        "dynamodb:ConditionCheckItem",
        "dynamodb:BatchWriteItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem",
        "dynamodb:DescribeTable"
       ],
       "Effect": "Allow",
       "Resource": [
        {
         "Fn::GetAtt": [
          "StateTable",
          "Arn"
         ]
        },
        {
         "Ref": "AWS::NoValue"
        }
       ]
      },
      {
       "Action": [
        "dynamodb:DeleteItem",
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query",
        "dynamodb:Scan",
        "dynamodb:BatchWriteItem",
        "dynamodb:UpdateItem"
       ],
       "Effect": "Allow",
       "Resource": [
        {
         "Fn::GetAtt": [
          "ConfigTable",
          "Arn"
         ]
        },
        {
         "Fn::GetAtt": [
          "MaintenanceWindowTable",
          "Arn"
         ]
        }
       ]
      },
      {
       "Action": [
        "ssm:PutParameter",
        "ssm:GetParameter"
       ],
       "Effect": "Allow",
       "Resource": {
        "Fn::Sub": "arn:${AWS::Partition}:ssm:${AWS::Region}:${AWS::AccountId}:parameter/Solutions/aws-instance-scheduler/UUID/*"
       }
      }
     ],
     "Version": "2012-10-17"
    },
    "PolicyName": "SchedulerRoleDefaultPolicy66F774B8",
    "Roles": [
     {
      "Ref": "SchedulerRole"
     }
    ]
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/SchedulerRole/DefaultPolicy/Resource",
    "cdk_nag": {
     "rules_to_suppress": [
      {
       "reason": "The scheduling lambda must access multiple resources across services",
       "id": "AwsSolutions-IAM5"
      }
     ]
    }
   }
  },
  "InstanceSchedulerEncryptionKey": {
   "Type": "AWS::KMS::Key",
   "Properties": {
    "KeyPolicy": {
     "Statement": [
      {
       "Action": "kms:*",
       "Effect": "Allow",
       "Principal": {
        "AWS": {
         "Fn::Join": [
          "",
          [
           "arn:",
           {
            "Ref": "AWS::Partition"
           },
           ":iam::",
           {
            "Ref": "AWS::AccountId"
           },
           ":root"
          ]
         ]
        }
       },
       "Resource": "*",
       "Sid": "default"
      },
      {
       "Action": [
        "kms:GenerateDataKey*",
        "kms:Decrypt"
       ],
       "Effect": "Allow",
       "Principal": {
        "AWS": {
         "Fn::GetAtt": [
          "SchedulerRole",
          "Arn"
         ]
        }
       },
       "Resource": "*",
       "Sid": "Allows use of key"
      }
     ],
     "Version": "2012-10-17"
    },
    "Description": "Key for SNS",
    "Enabled": true,
    "EnableKeyRotation": true
   },
   "UpdateReplacePolicy": "Delete",
   "DeletionPolicy": "Delete",
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/InstanceSchedulerEncryptionKey/Resource"
   }
  },
  "InstanceSchedulerEncryptionKeyAlias": {
   "Type": "AWS::KMS::Alias",
   "Properties": {
    "AliasName": {
     "Fn::Join": [
      "",
      [
       "alias/",
       {
        "Ref": "AWS::StackName"
       },
       "-instance-scheduler-encryption-key"
      ]
     ]
    },
    "TargetKeyId": {
     "Fn::GetAtt": [
      "InstanceSchedulerEncryptionKey",
      "Arn"
     ]
    }
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/InstanceSchedulerEncryptionKeyAlias/Resource"
   }
  },
  "InstanceSchedulerSnsTopic": {
   "Type": "AWS::SNS::Topic",
   "Properties": {
    "KmsMasterKeyId": {
     "Fn::GetAtt": [
      "InstanceSchedulerEncryptionKey",
      "Arn"
     ]
    }
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/InstanceSchedulerSnsTopic/Resource"
   }
  },
  "Main": {
   "Type": "AWS::Lambda::Function",
   "Properties": {
    "Code": {
     "S3Bucket": {
      "Fn::Sub": "solutions-${AWS::Region}"
     },
     "S3Key": "aws-instance-scheduler/v1.5.0/a9fbd49b1e2bd4f908ebff8ba450278da49ec8bf686be5b493b1a8c1942ae7c7.zip"
    },
    "Role": {
     "Fn::GetAtt": [
      "SchedulerRole",
      "Arn"
     ]
    },
    "Description": "EC2 and RDS instance scheduler, version 1.5.0",
    "Environment": {
     "Variables": {
      "SCHEDULER_FREQUENCY": {
       "Ref": "SchedulerFrequency"
      },
      "TAG_NAME": {
       "Ref": "TagName"
      },
      "LOG_GROUP": {
       "Ref": "SchedulerLogGroup"
      },
      "ACCOUNT": {
       "Ref": "AWS::AccountId"
      },
      "ISSUES_TOPIC_ARN": {
       "Ref": "InstanceSchedulerSnsTopic"
      },
      "STACK_NAME": {
       "Ref": "AWS::StackName"
      },
      "SEND_METRICS": {
       "Fn::FindInMap": [
        "mappings",
        "TrueFalse",
        {
         "Fn::FindInMap": [
          "Send",
          "AnonymousUsage",
          "Data"
         ]
        }
       ]
      },
      "SOLUTION_ID": {
       "Fn::FindInMap": [
        "mappings",
        "Settings",
        "MetricsSolutionId"
       ]
      },
      "TRACE": {
       "Fn::FindInMap": [
        "mappings",
        "TrueFalse",
        {
         "Ref": "Trace"
        }
       ]
      },
      "ENABLE_SSM_MAINTENANCE_WINDOWS": {
       "Fn::FindInMap": [
        "mappings",
        "TrueFalse",
        {
         "Ref": "EnableSSMMaintenanceWindows"
        }
       ]
      },
      "USER_AGENT": {
       "Fn::Join": [
        "",
        [
         "InstanceScheduler-",
         {
          "Ref": "AWS::StackName"
         },
         "-1.5.0"
        ]
       ]
      },
      "USER_AGENT_EXTRA": "AwsSolution/SO0030/1.5.0",
      "METRICS_URL": {
       "Fn::FindInMap": [
        "mappings",
        "Settings",
        "MetricsUrl"
       ]
      },
      "STACK_ID": {
       "Ref": "AWS::StackId"
      },
      "UUID_KEY": {
       "Fn::FindInMap": [
        "Send",
        "ParameterKey",
        "UniqueId"
       ]
      },
      "START_EC2_BATCH_SIZE": "5",
      "DDB_TABLE_NAME": {
       "Ref": "StateTable"
      },
      "CONFIG_TABLE": {
       "Ref": "ConfigTable"
      },
      "MAINTENANCE_WINDOW_TABLE": {
       "Ref": "MaintenanceWindowTable"
      },
      "STATE_TABLE": {
       "Ref": "StateTable"
      }
     }
    },
    "FunctionName": {
     "Fn::Join": [
      "",
      [
       {
        "Ref": "AWS::StackName"
       },
       "-InstanceSchedulerMain"
      ]
     ]
    },
    "Handler": "instance_scheduler.main.lambda_handler",
    "MemorySize": {
     "Ref": "MemorySize"
    },
    "Runtime": "python3.9",
    "Timeout": 300,
    "TracingConfig": {
     "Mode": "Active"
    }
   },
   "DependsOn": [
    "EC2DynamoDBPolicy",
    "Ec2PermissionsB6E87802",
    "SchedulerPolicy",
    "SchedulerRDSPolicy2E7C328A",
    "SchedulerRoleDefaultPolicy66F774B8",
    "SchedulerRole"
   ],
   "Metadata": {
    "cfn_nag": {
     "rules_to_suppress": [
      {
       "id": "W89",
       "reason": "This Lambda function does not need to access any resource provisioned within a VPC."
      },
      {
       "id": "W58",
       "reason": "This Lambda function has permission provided to write to CloudWatch logs using the iam roles."
      },
      {
       "id": "W92",
       "reason": "Lambda function is only used by the event rule periodically, concurrent calls are very limited."
      }
     ]
    }
   }
  },
  "StateTable": {
   "Type": "AWS::DynamoDB::Table",
   "Properties": {
    "KeySchema": [
     {
      "AttributeName": "service",
      "KeyType": "HASH"
     },
     {
      "AttributeName": "account-region",
      "KeyType": "RANGE"
     }
    ],
    "AttributeDefinitions": [
     {
      "AttributeName": "service",
      "AttributeType": "S"
     },
     {
      "AttributeName": "account-region",
      "AttributeType": "S"
     }
    ],
    "BillingMode": "PAY_PER_REQUEST",
    "PointInTimeRecoverySpecification": {
     "PointInTimeRecoveryEnabled": true
    },
    "SSESpecification": {
     "KMSMasterKeyId": {
      "Ref": "InstanceSchedulerEncryptionKey"
     },
     "SSEEnabled": true,
     "SSEType": "KMS"
    }
   },
   "UpdateReplacePolicy": "Delete",
   "DeletionPolicy": "Delete",
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/instance-scheduler-lambda/DynamoTable/Resource"
   }
  },
  "ConfigTable": {
   "Type": "AWS::DynamoDB::Table",
   "Properties": {
    "KeySchema": [
     {
      "AttributeName": "type",
      "KeyType": "HASH"
     },
     {
      "AttributeName": "name",
      "KeyType": "RANGE"
     }
    ],
    "AttributeDefinitions": [
     {
      "AttributeName": "type",
      "AttributeType": "S"
     },
     {
      "AttributeName": "name",
      "AttributeType": "S"
     }
    ],
    "BillingMode": "PAY_PER_REQUEST",
    "PointInTimeRecoverySpecification": {
     "PointInTimeRecoveryEnabled": true
    },
    "SSESpecification": {
     "KMSMasterKeyId": {
      "Ref": "InstanceSchedulerEncryptionKey"
     },
     "SSEEnabled": true,
     "SSEType": "KMS"
    }
   },
   "UpdateReplacePolicy": "Delete",
   "DeletionPolicy": "Delete",
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/ConfigTable/Resource"
   }
  },
  "MaintenanceWindowTable": {
   "Type": "AWS::DynamoDB::Table",
   "Properties": {
    "KeySchema": [
     {
      "AttributeName": "Name",
      "KeyType": "HASH"
     },
     {
      "AttributeName": "account-region",
      "KeyType": "RANGE"
     }
    ],
    "AttributeDefinitions": [
     {
      "AttributeName": "Name",
      "AttributeType": "S"
     },
     {
      "AttributeName": "account-region",
      "AttributeType": "S"
     }
    ],
    "BillingMode": "PAY_PER_REQUEST",
    "PointInTimeRecoverySpecification": {
     "PointInTimeRecoveryEnabled": true
    },
    "SSESpecification": {
     "KMSMasterKeyId": {
      "Ref": "InstanceSchedulerEncryptionKey"
     },
     "SSEEnabled": true,
     "SSEType": "KMS"
    }
   },
   "UpdateReplacePolicy": "Delete",
   "DeletionPolicy": "Delete",
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/MaintenanceWindowTable/Resource"
   }
  },
  "schedulereventbus": {
   "Type": "AWS::Events::EventBus",
   "Properties": {
    "Name": {
     "Fn::Join": [
      "",
      [
       {
        "Ref": "Namespace"
       },
       "-",
       {
        "Fn::FindInMap": [
         "mappings",
         "SchedulerEventBusName",
         "Name"
        ]
       }
      ]
     ]
    }
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/scheduler-event-bus"
   },
   "Condition": "IsMemberOfOrganization"
  },
  "schedulereventbuspolicy": {
   "Type": "AWS::Events::EventBusPolicy",
   "Properties": {
    "StatementId": {
     "Fn::GetAtt": [
      "schedulereventbus",
      "Name"
     ]
    },
    "Action": "events:PutEvents",
    "Condition": {
     "Key": "aws:PrincipalOrgID",
     "Type": "StringEquals",
     "Value": {
      "Fn::Select": [
       0,
       {
        "Ref": "Principals"
       }
      ]
     }
    },
    "EventBusName": {
     "Fn::GetAtt": [
      "schedulereventbus",
      "Name"
     ]
    },
    "Principal": "*"
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/scheduler-event-bus-policy"
   },
   "Condition": "IsMemberOfOrganization"
  },
  "schedulerssmparametercrossaccountevents": {
   "Type": "AWS::Events::Rule",
   "Properties": {
    "Description": "Event rule to invoke Instance Scheduler lambda function to store spoke account id(s) in configuration.",
    "EventBusName": {
     "Fn::GetAtt": [
      "schedulereventbus",
      "Name"
     ]
    },
    "EventPattern": {
     "source": [
      "aws.ssm"
     ],
     "detail-type": [
      "Parameter Store Change"
     ],
     "detail": {
      "name": [
       "/instance-scheduler/do-not-delete-manually"
      ],
      "operation": [
       "Create",
       "Delete"
      ],
      "type": [
       "String"
      ]
     }
    },
    "State": "ENABLED",
    "Targets": [
     {
      "Arn": {
       "Fn::GetAtt": [
        "Main",
        "Arn"
       ]
      },
      "Id": "Scheduler-Lambda-Function"
     }
    ]
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/scheduler-ssm-parameter-cross-account-events"
   },
   "Condition": "IsMemberOfOrganization"
  },
  "EventBusRuleLambdaPermission": {
   "Type": "AWS::Lambda::Permission",
   "Properties": {
    "Action": "lambda:InvokeFunction",
    "FunctionName": {
     "Ref": "Main"
    },
    "Principal": "events.amazonaws.com",
    "SourceArn": {
     "Fn::GetAtt": [
      "schedulerssmparametercrossaccountevents",
      "Arn"
     ]
    }
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/EventBusRuleLambdaPermission"
   },
   "Condition": "IsMemberOfOrganization"
  },
  "SchedulerRule": {
   "Type": "AWS::Events::Rule",
   "Properties": {
    "Description": "Instance Scheduler - Rule to trigger instance for scheduler function version 1.5.0",
    "ScheduleExpression": {
     "Fn::FindInMap": [
      "mappings",
      "Timeouts",
      {
       "Ref": "SchedulerFrequency"
      }
     ]
    },
    "State": {
     "Fn::FindInMap": [
      "mappings",
      "EnabledDisabled",
      {
       "Ref": "SchedulingActive"
      }
     ]
    },
    "Targets": [
     {
      "Arn": {
       "Fn::GetAtt": [
        "Main",
        "Arn"
       ]
      },
      "Id": "Target0"
     }
    ]
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/SchedulerEventRule/Resource"
   }
  },
  "SchedulerEventRuleAllowEventRuleawsinstanceschedulerschedulerlambda711BB00B6BF6E2FE": {
   "Type": "AWS::Lambda::Permission",
   "Properties": {
    "Action": "lambda:InvokeFunction",
    "FunctionName": {
     "Fn::GetAtt": [
      "Main",
      "Arn"
     ]
    },
    "Principal": "events.amazonaws.com",
    "SourceArn": {
     "Fn::GetAtt": [
      "SchedulerRule",
      "Arn"
     ]
    }
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/SchedulerEventRule/AllowEventRuleawsinstanceschedulerschedulerlambda711BB00B"
   }
  },
  "SchedulerConfigHelper": {
   "Type": "Custom::ServiceSetup",
   "Properties": {
    "ServiceToken": {
     "Fn::GetAtt": [
      "Main",
      "Arn"
     ]
    },
    "timeout": 120,
    "config_table": {
     "Ref": "ConfigTable"
    },
    "tagname": {
     "Ref": "TagName"
    },
    "default_timezone": {
     "Ref": "DefaultTimezone"
    },
    "use_metrics": {
     "Fn::FindInMap": [
      "mappings",
      "TrueFalse",
      {
       "Ref": "UseCloudWatchMetrics"
      }
     ]
    },
    "scheduled_services": {
     "Fn::Split": [
      ",",
      {
       "Fn::FindInMap": [
        "mappings",
        "Services",
        {
         "Ref": "ScheduledServices"
        }
       ]
      }
     ]
    },
    "schedule_clusters": {
     "Fn::FindInMap": [
      "mappings",
      "TrueFalse",
      {
       "Ref": "ScheduleRdsClusters"
      }
     ]
    },
    "create_rds_snapshot": {
     "Fn::FindInMap": [
      "mappings",
      "TrueFalse",
      {
       "Ref": "CreateRdsSnapshot"
      }
     ]
    },
    "regions": {
     "Ref": "Regions"
    },
    "remote_account_ids": {
     "Ref": "Principals"
    },
    "namespace": {
     "Ref": "Namespace"
    },
    "aws_partition": {
     "Fn::Sub": "${AWS::Partition}"
    },
    "scheduler_role_name": {
     "Fn::FindInMap": [
      "mappings",
      "SchedulerRole",
      "Name"
     ]
    },
    "schedule_lambda_account": {
     "Fn::FindInMap": [
      "mappings",
      "TrueFalse",
      {
       "Ref": "ScheduleLambdaAccount"
      }
     ]
    },
    "trace": {
     "Fn::FindInMap": [
      "mappings",
      "TrueFalse",
      {
       "Ref": "Trace"
      }
     ]
    },
    "enable_SSM_maintenance_windows": {
     "Fn::FindInMap": [
      "mappings",
      "TrueFalse",
      {
       "Ref": "EnableSSMMaintenanceWindows"
      }
     ]
    },
    "log_retention_days": {
     "Ref": "LogRetentionDays"
    },
    "started_tags": {
     "Ref": "StartedTags"
    },
    "stopped_tags": {
     "Ref": "StoppedTags"
    },
    "stack_version": "1.5.0",
    "use_aws_organizations": {
     "Fn::FindInMap": [
      "mappings",
      "TrueFalse",
      {
       "Ref": "UsingAWSOrganizations"
      }
     ]
    }
   },
   "DependsOn": [
    "SchedulerLogGroup"
   ],
   "UpdateReplacePolicy": "Delete",
   "DeletionPolicy": "Delete",
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/ServiceSetup/Default"
   }
  },
  "Ec2PermissionsB6E87802": {
   "Type": "AWS::IAM::Policy",
   "Properties": {
    "PolicyDocument": {
     "Statement": [
      {
       "Action": "ec2:ModifyInstanceAttribute",
       "Effect": "Allow",
       "Resource": {
        "Fn::Sub": "arn:${AWS::Partition}:ec2:*:${AWS::AccountId}:instance/*"
       }
      },
      {
       "Action": "sts:AssumeRole",
       "Effect": "Allow",
       "Resource": {
        "Fn::Sub": [
         "arn:${AWS::Partition}:iam::*:role/${Namespace}-${Name}",
         {
          "Name": {
           "Fn::FindInMap": [
            "mappings",
            "SchedulerRole",
            "Name"
           ]
          }
         }
        ]
       }
      }
     ],
     "Version": "2012-10-17"
    },
    "PolicyName": "Ec2PermissionsB6E87802",
    "Roles": [
     {
      "Ref": "SchedulerRole"
     }
    ]
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/Ec2Permissions/Resource",
    "cdk_nag": {
     "rules_to_suppress": [
      {
       "reason": "This Lambda function needs to be able to modify ec2 instances for scheduling purposes.",
       "id": "AwsSolutions-IAM5"
      }
     ]
    }
   }
  },
  "EC2DynamoDBPolicy": {
   "Type": "AWS::IAM::Policy",
   "Properties": {
    "PolicyDocument": {
     "Statement": [
      {
       "Action": [
        "ssm:GetParameter",
        "ssm:GetParameters"
       ],
       "Effect": "Allow",
       "Resource": {
        "Fn::Sub": "arn:${AWS::Partition}:ssm:*:${AWS::AccountId}:parameter/*"
       }
      },
      {
       "Action": [
        "logs:DescribeLogStreams",
        "rds:DescribeDBClusters",
        "rds:DescribeDBInstances",
        "ec2:DescribeInstances",
        "ec2:DescribeRegions",
        "cloudwatch:PutMetricData",
        "ssm:DescribeMaintenanceWindows",
        "tag:GetResources"
       ],
       "Effect": "Allow",
       "Resource": "*"
      },
      {
       "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:PutRetentionPolicy"
       ],
       "Effect": "Allow",
       "Resource": [
        {
         "Fn::Sub": "arn:${AWS::Partition}:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/*"
        },
        {
         "Fn::GetAtt": [
          "SchedulerLogGroup",
          "Arn"
         ]
        }
       ]
      }
     ],
     "Version": "2012-10-17"
    },
    "PolicyName": "EC2DynamoDBPolicy",
    "Roles": [
     {
      "Ref": "SchedulerRole"
     }
    ]
   },
   "Metadata": {
    "cfn_nag": {
     "rules_to_suppress": [
      {
       "id": "W12",
       "reason": "All policies have been scoped to be as restrictive as possible. This solution needs to access ec2/rds resources across all regions."
      }
     ]
    },
    "cdk_nag": {
     "rules_to_suppress": [
      {
       "reason": "All policies have been scoped to be as restrictive as possible. This solution needs to access ec2/rds resources across all regions.",
       "id": "AwsSolutions-IAM5"
      }
     ]
    }
   }
  },
  "SchedulerPolicy": {
   "Type": "AWS::IAM::Policy",
   "Properties": {
    "PolicyDocument": {
     "Statement": [
      {
       "Action": [
        "rds:AddTagsToResource",
        "rds:RemoveTagsFromResource",
        "rds:DescribeDBSnapshots",
        "rds:StartDBInstance",
        "rds:StopDBInstance"
       ],
       "Effect": "Allow",
       "Resource": {
        "Fn::Sub": "arn:${AWS::Partition}:rds:*:${AWS::AccountId}:db:*"
       }
      },
      {
       "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:CreateTags",
        "ec2:DeleteTags"
       ],
       "Effect": "Allow",
       "Resource": {
        "Fn::Sub": "arn:${AWS::Partition}:ec2:*:${AWS::AccountId}:instance/*"
       }
      },
      {
       "Action": "sns:Publish",
       "Effect": "Allow",
       "Resource": {
        "Ref": "InstanceSchedulerSnsTopic"
       }
      },
      {
       "Action": "lambda:InvokeFunction",
       "Effect": "Allow",
       "Resource": {
        "Fn::Sub": "arn:${AWS::Partition}:lambda:${AWS::Region}:${AWS::AccountId}:function:${AWS::StackName}-InstanceSchedulerMain"
       }
      },
      {
       "Action": [
        "kms:GenerateDataKey*",
        "kms:Decrypt"
       ],
       "Effect": "Allow",
       "Resource": {
        "Fn::GetAtt": [
         "InstanceSchedulerEncryptionKey",
         "Arn"
        ]
       }
      }
     ],
     "Version": "2012-10-17"
    },
    "PolicyName": "SchedulerPolicy",
    "Roles": [
     {
      "Ref": "SchedulerRole"
     }
    ]
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/SchedulerPolicy/Resource",
    "cdk_nag": {
     "rules_to_suppress": [
      {
       "reason": "All policies have been scoped to be as restrictive as possible. This solution needs to access ec2/rds resources across all regions.",
       "id": "AwsSolutions-IAM5"
      }
     ]
    }
   }
  },
  "SchedulerRDSPolicy2E7C328A": {
   "Type": "AWS::IAM::Policy",
   "Properties": {
    "PolicyDocument": {
     "Statement": [
      {
       "Action": [
        "rds:DeleteDBSnapshot",
        "rds:DescribeDBSnapshots",
        "rds:StopDBInstance"
       ],
       "Effect": "Allow",
       "Resource": {
        "Fn::Sub": "arn:${AWS::Partition}:rds:*:${AWS::AccountId}:snapshot:*"
       }
      },
      {
       "Action": [
        "rds:AddTagsToResource",
        "rds:RemoveTagsFromResource",
        "rds:StartDBCluster",
        "rds:StopDBCluster"
       ],
       "Effect": "Allow",
       "Resource": {
        "Fn::Sub": "arn:${AWS::Partition}:rds:*:${AWS::AccountId}:cluster:*"
       }
      }
     ],
     "Version": "2012-10-17"
    },
    "PolicyName": "SchedulerRDSPolicy2E7C328A",
    "Roles": [
     {
      "Ref": "SchedulerRole"
     }
    ]
   },
   "Metadata": {
    "aws:cdk:path": "aws-instance-scheduler/SchedulerRDSPolicy/Resource",
    "cdk_nag": {
     "rules_to_suppress": [
      {
       "reason": "All policies have been scoped to be as restrictive as possible. This solution needs to access ec2/rds resources across all regions.",
       "id": "AwsSolutions-IAM5"
      }
     ]
    }
   }
  }
 },
 "Outputs": {
  "AppRegistryApplicationManagerUrl775D5C3D": {
   "Description": "Application manager url for the application created.",
   "Value": {
    "Fn::Join": [
     "",
     [
      "https://",
      {
       "Ref": "AWS::Region"
      },
      ".console.aws.amazon.com/systems-manager/appmanager/application/AWS_AppRegistry_Application-",
      {
       "Fn::Join": [
        "-",
        [
         {
          "Fn::FindInMap": [
           "AppRegistryForInstanceSchedulerSolution25A90F05",
           "Data",
           "AppRegistryApplicationName"
          ]
         },
         {
          "Ref": "AWS::Region"
         },
         {
          "Ref": "AWS::AccountId"
         },
         {
          "Ref": "AWS::StackName"
         }
        ]
       ]
      }
     ]
    ]
   }
  },
  "AccountId": {
   "Description": "Account to give access to when creating cross-account access role for cross account scenario ",
   "Value": {
    "Ref": "AWS::AccountId"
   }
  },
  "ConfigurationTable": {
   "Description": "Name of the DynamoDB configuration table",
   "Value": {
    "Fn::GetAtt": [
     "ConfigTable",
     "Arn"
    ]
   }
  },
  "IssueSnsTopicArn": {
   "Description": "Topic to subscribe to for notifications of errors and warnings",
   "Value": {
    "Ref": "InstanceSchedulerSnsTopic"
   }
  },
  "SchedulerRoleArn": {
   "Description": "Role for the instance scheduler lambda function",
   "Value": {
    "Fn::GetAtt": [
     "SchedulerRole",
     "Arn"
    ]
   }
  },
  "ServiceInstanceScheduleServiceToken": {
   "Description": "Arn to use as ServiceToken property for custom resource type Custom::ServiceInstanceSchedule",
   "Value": {
    "Fn::GetAtt": [
     "Main",
     "Arn"
    ]
   }
  }
 }
}
```
:::

# CloudFormation実行
テンプレートファイルをアップロードし、スタック作成を進めていくと以下のパラメータ入力が求められます。

#### Scheduler
- TagName
  - インスタンススケジューラー対象となるタグの定義付け(デフォルトでScheduleが付与)
- ScheduledServices
  - 対象のAWSサービス(EC2、RDS、その両方の中から選択)
- ScheduleRDSClusters
  - Auroraクラスターのスケジューラーを有効可否
- CreateRDSSnapshot
  - 停止前にRDSのスナップショット取得可否
- SchedulingActive
  - スケジューラーの即時有効可否
- Regions
  - 対象リージョン選択
- DefaultTimezone
  - デフォルトのタイムゾーン設定
- CrossAccountRoles
  - クロスアカウントロール
- This account
  - 現在のアカウントのリソースに対してスケジューラーの有効可否
- SchedulerFrequency
  - Lambdaの実行頻度(分単位)
- MemorySize
  - Lambdaのメモリサイズ指定

#### Option
- UseCloudWatchMetrics
  - CloudWatchメトリクスによるインスタンスデータ収集の有効可否
- UseCloudWatchLogs
  - CloudWatchによるロギング設定の有効可否
- EnableSSMMaintenanceWindows
  - SSMメンテナンスウィンドウをロードしてEC2のスケジューラーの設定有効可否

#### Other parameters
- LogRetentionDays
  - ログ保持期間
- StartedTags
  - 起動したインスタンスに追加するタグ
- StoppedTags
  - 停止したインスタンスに追加するタグ

基本的にデフォルト設定のままで問題ありません。
EC2インスタンスとRDSインスタンスの両方をインスタンススケジューラー対象にしたい場合、`Both`を選択します。
そのほかですと、Regionで東京を希望する場合は`ap-northeast-1`、日本時間でスケジューリングしたい場合はTimeZoneを`Asia/Tokyo`にします。
パラメータ入力を完了し、スタック作成の確認画面まで進むとIAMの新規作成のチェック確認がありますので忘れずにチェックします。
![](/images/ec2-schedule/image2.png)

これを起動することで上記図のアーキテクチャが構築されEC2(またはRDS)にTagに`Schedule`をつけたものが対応したスケジュールアクションを設定することで自動で停止起動が実現できます。

# サンプルハンズオン
起動できたらまずはDynamoDBを見てみます。
いくつかテーブルが作られていると思いますが、`ConfigTable`を見てます。
![](/images/ec2-schedule/image3.png)
注目するポイントはTypeの`period`と`schedule`です。
##### period
インスタンスの稼働時間を指定できます。
サンプルで作成された`office-hours`は平日月曜日~金曜日の10:00~19:00の間に稼働することを意味しています。

| フィールド| 値 |
| ---- | ---- |
| begintime | 10:00 |
| endtime | 19:00 |
| name | office-hours|
| weekdays | mon-fri |

![](/images/ec2-schedule/image4.png)

##### schedule
scheduleに複数のperiodをまとめることでインスタンスに複数のperiodのスケジューリング設定を反映させることができます。
サンプルである`jp-office-hours`はJST基準で`office-hours`のperiod設定を反映させる設定になります。

| フィールド| 値 |
| ---- | ---- |
| period | office-hours |
| timezone | Asia/Tokyo |
| name | jp-office-hours |

![](/images/ec2-schedule/image5.png)

このタグを稼働しているインスタンスに付与します。
このハンズオンを実施している時間帯は日曜日ですので、スケジュールタグ`Schedule`を付与したらEC2は自動で停止するようになります。
![](/images/ec2-schedule/image6.png)
*タグ付与直後*

![](/images/ec2-schedule/image7.png)
*数分後*

タグを付与してから数分後にEC2のステータスが変更し、停止されました。
ここで土曜日、日曜日は終日起動するというperiod`weekends`を`jp-office-hours`のscheduleに追加します。
![](/images/ec2-schedule/image8.png)
![](/images/ec2-schedule/image9.png)

テーブルの編集が完了したらあっという間にEC2が起動されました。
![](/images/ec2-schedule/image10.png)

このようにscheduleに複数のperiod設定をつけた場合どれか1つの期間を満たせばEC2が起動するようになっています。
これを活用することで例えば9:00-12:00の間起動し、12:00-13:00の間は停止、また13:00から起動するといった細やかな設定も可能になります。

# Scheduler CLIによるスケジューリング設定
スケジュールの設定は前述のとおりDynamoDBのテーブルを編集することで新規にスケジュールを作成できますが、Instance Schedulerには専用のCLIコマンドも用意されています。
次にCLIコマンドを活用したスケジューリング設定について説明します。

## インストール方法
下記リンクからScheduler CLIパッケージをダウンロードしてローカルに展開します。
https://s3.amazonaws.com/solutions-reference/aws-instance-scheduler/latest/scheduler-cli.zip

展開したらディレクトリ`scheduler-cli`へ移動して`setup.py`を実行します。

```bash
Python setup.py install

# インストール完了後確認
scheduler-cli --version
scheduler-cli v1.2.0
```

## 使い方
コマンドの構造としてサブコマンドと変数指定があります。

```bash
scheduler-cli <sub-command> <arguments>
```

サブコマンドには以下のものがあります。

- create-period       
- create-schedule     
- delete-period       
- delete-schedule     
- describe-periods    
- describe-schedule-usage
- describe-schedules  
- update-period       
- update-schedule     

CLIコマンドを使ってperiodとscheduleを新規に作成していきます。

#### period作成

```bash
scheduler-cli create-period --name "sample-period" --begintime 10:00 --endtime 15:00 --weekdays mon-fri --stack test-schedule
{
   "Period": {
      "Begintime": "10:00",
      "Endtime": "15:00",
      "Name": "sample-period",
      "Weekdays": [
         "mon-fri"
      ],
      "Type": "period"
   }
}
```

![](/images/ec2-schedule/image11.png)

#### schedule作成

```bash
 scheduler-cli create-schedule --name sample-schedule --periods sample-period --timezone Asia/Tokyo --stack test-schedule
{
   "Schedule": {
      "Timezone": "Asia/Tokyo",
      "Name": "sample-schedule",
      "Periods": [
         "sample-period"
      ],
      "StopNewInstances": true,
      "UseMaintenanceWindow": false,
      "RetainRunning": false,
      "Enforced": false,
      "Hibernate": false,
      "UseMetrics": false,
      "Type": "schedule"
   }
}
```

![](/images/ec2-schedule/image12.png)

:::message
--stackオプションですが、ここではCloudFormationで作成したスタックの名前を指定してください。
:::

コマンド実行してDynamoDBのテーブルにもCLIで作成したperiodとscheduleが表示されていることを確認できました。これでCLIによるインスタンスのスケジューリング設定も行なえることができます。

# 追記(2021/9/17)
Instance Schedulerの設定項目でクロスアカウントロールの設定がありました。これを活用すればマルチアカウント運用時でも一環境のスタックから他のアカウントのEC2インスタンスやRDSにもスケジューラー設定を実装できることが分かりましたので記載いたします。

## リモートアカウントでのスケジューラー設定
こちらのリンクからリモートアカウントで実行するためのスタックテンプレートファイルをダウンロードします。

https://s3.amazonaws.com/solutions-reference/aws-instance-scheduler/latest/aws-instance-scheduler-remote.template

CloudFormationにアップロードさせましたらInstance Schedulerが動いているAWSアカウントの番号を入力します。
![](/images/ec2-schedule/image13.png)

実行後、出力タブを確認するとクロスアカウントロールが作成されます。
値のロールARNを本家アカウントで使用しますので控えておきます。
![](/images/ec2-schedule/image14.png)

本家アカウントのCloudFormationですでに実行したスタックの更新を行ないます。
[CloudFormation実行](https://zenn.dev/yuta28/articles/ec2-schedule#cloudformation%E5%AE%9F%E8%A1%8C)で触れてますが、クロスアカウントロールを記入する箇所がありますのでここに控えたロールARNを記入します。
![](/images/ec2-schedule/image15.png)

更新して`UPDATE_COMPLETE`になりましたらリモートアカウント先のEC2インスタンスにスケジューラータグを設定して確認してみます。
リモートアカウント先でEC2インスタンスを作成し、スケジューラータグをつけます。
![](/images/ec2-schedule/image16.png)
sample-scheduleには以下のスケジュールが設定されています。


```json:sample-period
{
  "type": {
    "S": "period"
  },
  "name": {
    "S": "sample-period"
  },
  "begintime": {
    "S": "17:44"
  },
  "endtime": {
    "S": "17:48"
  },
  "weekdays": {
    "SS": [
      "sat-sun"
    ]
  }
}
```

土日の17:44~17:48の間に稼働するという設定です。
EC2の作成時刻は17:40辺りですので少し待ってみて自動で起動できることを確認します。

##### 5分後
ちゃんと起動されていました。
![](/images/ec2-schedule/image17.png)
さらに待って停止も実行されるか見てみます。
##### さらに5分後
![](/images/ec2-schedule/image18.png)
ご覧の通り停止されていました。
EC2インスタンス画面だけでは分かりにくいと思いましたのでCloudTrailでも結果を見てみます。
##### CloudTrail
![](/images/ec2-schedule/image19.png)
CloudTrailで確認するとEC2インスタンスの起動停止の実行ユーザーが`ec2-scheduler-アカウント番号`になっていました。

このようにマルチアカウントでAWSを運用しているからといってすべてのAWSアカウントにInstance Schedulerを設定する必要はなく、最初に導入したアカウントからクロスアカウントロールで別のAWSアカウントにスケジューラー設定を反映させることができます。
# 所感
Instance Schedulerを使ってEC2の稼働時間の管理ができました。
検証環境は基本的に深夜早朝、土日は動かす必要はないので停止したほうがいいですが、毎回仕事終わりにEC2を停止する運用にしても忘れることもありますし、検証環境の数が多いと一々手で止めるなんてことを行なうとオペレーションコストが高まってしまいます。
Instance SchedulerでEC2のコスト最適化を推進して無駄を減らしていこうと思います。

# LT資料
LTのスライド資料です。
@[speakerdeck](4e5899a5aeae4fe18d8bc7ebc38db918)
# 参考文献
https://d1.awsstatic.com/Solutions/ja_JP/instance-scheduler.pdf

https://www.youtube.com/watch?v=HqXMxdjv638

https://docs.aws.amazon.com/solutions/latest/instance-scheduler/scheduler-cli.html