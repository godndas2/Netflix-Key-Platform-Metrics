# Netflix-Key-Platform-Metrics

# Assumptions

- Total number of daily active users = 100 million
- The peak daily active users, 100 million * 3 = 300 million
- The max peak daily active users in 3 months, 300 million * 2 = 600 million
- The average number of videos watched by each user per day = 5
- The average size of one video = 500 MB
- The average number of videos uploaded per day from the backend = 1,000
- Total number of videos watched per day = 100 million *5 =500 million
- Total peak video per day = 1.5 billion
- Total max peak video per day = 3 billion
- Total egress per day = 500 million * 500 MB = 250 PB (Peta Byte)
- Egress bandwidth = 29.1GB/sec
- Total ingress for upload = 1,000 * 500MB = 500GB
- Ingress bandwidth =5.8MB/sec
- Total Storage required in 5 years = 500 GB*5*365 = 912.5TB （please note that Netflix creates multiple formats and resolutions for each video optimized for different device types. So the storage will be more than 912.5TB.

![image](https://user-images.githubusercontent.com/34512538/197699207-9d15bcb1-5c0b-4f0c-99ad-b81587ddd714.png)
![image](https://user-images.githubusercontent.com/34512538/197699280-bb22bbe9-12a0-448a-83e8-86f6234d3883.png)
![image](https://user-images.githubusercontent.com/34512538/197699294-15cf8455-e916-4a50-8681-e420b6760ed6.png)


# Backend- Services

- User and authentication service (Mainly responsible for user authentication and profiles. The data will be in a relational database like MySQL or PostgreSQL. SO, RDBMS is a suitable choice.
- Subscription Management service (Manage the subscription of the users. Since data processing by this service is highly transactional in nature, RDBMS is a suitable choice.
- Videos Service (Surfacing videos to end-users. This service store videos metadata in an RDBMS like MySQL or PostgreSQL. For quick response time, this service would implement a write-around cache using an in-memory cache like Redis or Memcached.
- TransCoder Service (Checking the quality of uploaded videos, compressing the videos with different codecs, creating different resolutions of the video. Once a video is uploaded to the Transcoder service, it will upload the same to an internal distributed storage like an S3 bucket and add an entry to the database. Kafka or RabbitMQ processes a message in the queue. So, the backend workers would consume messages from the queue, download the video from the internal S3 bucket and transcode it to different formats. Once transcoding is complete, the worker would upload the video to an external S3 bucket and update the status of the video in the database as active for viewing by end-users. The worker would also add an entry of the video metadata in the search database which supports full-text search. This would enable the end-users to search for videos using their title or summary. Then, the video from an external S3 bucket would also be cached over a CDN for reducing latency and improving playback performance.
- Global Search Service (enable end-users to search for a video using metadata like title, summary, etc. Elasticsearch or Solr can be used to support full-text searching because they are stored in Elastic Search DB which enables users to search for movies, series by title, or any meta-data associated with the video. This service also ranks the results based on recency, reviews, recommendations, and popularity for a better user experience. Also, the elastic search can track down users’ events in cases of failure. Then the customer care team uses elastic search to resolve issues.
