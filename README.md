# Rails Dynamic Request

## Objectives


1. Explain the differences between a static route and a dynamic route with regards to variables
2. Write a dynamic route with the :variable syntax
3. Access a route variable value from params
4. Use a route variable value in params within a controller action (like in Post.find(params[:id])
5. Use the convention of naming the primary key :id as a route variable
6. Pass an instance variable to the view
7. Use a singular AR instance in a view to generate a dynamic template based on data loaded via a route variable * (this objective should be re-written)* 

## Notes

The next restful action we want to build is the ability to show a post.

to do that we need to integrate route variables into our routes for a post /posts/1 can't be hardcoded.

how to draw a dynamic route, :variable syntax and how that is arbitrary but will correspond to params.

how to access the dynamic route variable in a controller action and param how to use that data in find Post

passing an instance variable to a view
