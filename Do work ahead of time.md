# Summary
- prepare ingredients before dinner shift starts.
	- Don't run across your kitchen ([[CPU cache]], [[DRAM]] or even disk) when the work begins, prepare whats needed for this task ahead of time and utilize it fully ([[One task at a time, many repetitions]]). 
	- Keep data ready and tight ([[d-cache]], [[i-cache]])
- precompute and bake as many things as possible. Do the hard and time consuming time ahead of when it already needs to be used. (point about heavy data)
	- Calculate heavy thing early and reuse it
- don's start making croissants from scratch when customer order one. Get a lot of things ready and in place, if you know that you need to process a lot of them. (point about heavy pipeline)
	- Don't trigger multi-step workflow or heavy pipelines in hot-paths which involves [[Page Fault]], 
	- If you know you'll need N things, get N things. avoid dynamic work


# Techniques
- [[object pool]]
- [[data flattening]]
