[Datacube.io](https://datacube.io) is a SaaS backup service for startups. It lets you set up backups for all your servers, databases, and even git repositories, in just a few minutes.

Here is our architecture.

### Technology stack

Primary languages:

- Ruby for the front-end
- Shell and Ruby for the back-end

Full stack:

- Bootstrap 3
- Ruby on Rails 4 (Amazon Elastic Beanstalk)
- Postgres (Amazon Relational Database Service)
- Docker
- Message queues (Amazon Simple Queue Service)
- Ubuntu (RunAbove Instances)
- OpenStack Swift (RunAbove Object Storage)

### General architecture

Our services were initially all in AWS. We decided to move our main servers and the object storage to [RunAbove](https://www.runabove.com), to be less far from our users (who are mainly in France) and benefit from better performances.

![](http://i.imgur.com/OiyvmQ6.png)

RunAbove being built on OpenStack, it also has the advantage to make us less vendor-dependant.

### How does it work?

#### Front-end

- **Web application:** This interface is built with Bootstrap and Ruby on Rails. It lets our users create, manage, and download backups. It's hosted on Amazon Elastic Beanstalk to ensure scalability and high availability.

#### Back-end

- **Database:** A Postgres, high-availability, Amazon RDS. All the backup information is encrypted. As Datacube.io is a write-heavy system, the performances are better than with MySQL.
- **Task server:** Every hour, this server written in Ruby reads from the database the backups that must be done. The information is then pushed into a queue.
- **Queues:** There are different queues, all based on Amazon SQS, for different types of data (`downloads`, `backups`, `deletes`, ...).
- **Workers:** AKA backup servers. These RunAbove instances are managed with Docker and Docker Machine. The application running on it is written in Ruby and Shell. It's continuously pulling from the queues and do the actual backups. It's possible to increase or decrease the number of instances in 5 minutes, without reconfiguration.

#### Backup process

This is what happens when Datacube.io starts backing up data from a customer's server:

1. Each file is **split into blocks** of 8MB each.
* To save space and improve performances, a **global deduplication** process has been implemented. It means that if a block has already been stored in the past, Datacube.io deletes it and moves on to the next block. Our deduplication ratio is in the two digits.
* The blocks are **compressed** using a high-performance algorithm to save space.
* Then, the blocks are **encrypted** with AES-256bits to secure the data.
* Finally, the blocks are uploaded to the Swift storage.

![](http://i.imgur.com/Zje3c7m.png)

To have great performances, this Ruby application is **multi-threaded** so that this process can be done on multiple blocks at the same time.

### Next challenges

- High availability for the task server.
- **Easier code deployment** to the workers (although the process is based on Docker, some actions are not automated).
- Being able to **simulate workloads** for the workers.
- **Variable block size** to improve the deduplication ratio.