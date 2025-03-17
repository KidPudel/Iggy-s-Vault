Each [[consumer]] in a group doesn't share a [[partition in kafka]], meaning each consumer read different messages

Have multiple groups is useful when we have different applications reading from the same producer
For each consumer in a group will have their own [[offset in partion]]