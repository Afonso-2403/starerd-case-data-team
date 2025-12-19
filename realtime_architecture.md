# Real Time Architecture Ideas

## Dataflow Diagram

A sketch of a possible real time architecture was done with [excalidraw](https://excalidraw.com/). The file `real_time_architecture_sketch.excalidraw` can be uploaded to the browser for iteration and the most up to date picture is included in the repo as well.

The idea is the ingestion of the survey data through an API that forwards it to some kind of message broker. The cleaning and validation service then fetches the data from the broker. It can either be with a publish/subscribe mechanism for streaming or through a frequent pull by the cleaning service for micro-batch processing. 

Given the requirement of available analytics within seconds of submission, I would assume that a streaming approach would be needed. I have only worked with micro-batch designs, I would like to hear your thoughts on possible streaming approaches.

I have read about the concept of a Dead Letter Queue to handle failures in delivering/processing of data, but I am not familiar with it, I would like to hear your thoughts on using it to have a more robust system.

The cleaning and validation service would interact with the data storage to store the raw data and insert/update user metadata. It would also log statistics on missing/defective data to a metrics service for monitoring.  

The staging data would then be handled by the processing service (this is sketched as different from the cleaning and validation service but they could "live" in the same place given their tight coupling, much like the implemented batch approach), that would create the fact table and compute aggregations, while logging processing statistics.  
In the sketch there is a mention to updating rolling aggregates, I have used this in a micro-batch architecture before with good results, in a streaming approach I am unsure if it would be optimal.

The downstream analytics layer will be responsive to updates in the relevant aggregated data through some kind of event-based system.

I have only included one data storage in the architecture but would like to hear your thoughts on using multiple data storing services to optimise for analytics, for example.  