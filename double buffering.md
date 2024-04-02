This technique is used to avoid flickering and tearing.
It works by having two buffers, front and back buffers.
The process:
- When rending a frame, we have front buffer, that has already rendered data inside it (previous frame), that is used to display a frame, meanwhile [[graphics pipeline]] uses back buffer to render a new frame into it.
- Once the rendering into a back buffer completed, front buffer is switched with a back buffer and now back buffer is the front buffer and stored data is displayed on the screen
- Now [[graphics pipeline]] rendering new frame into a newly named back buffer