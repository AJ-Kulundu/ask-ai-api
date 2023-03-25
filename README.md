## System dependencies

The app has the following dependencies:

- Ruby 3.1.3
- Rails 7.0.4.3
- PostgreSQL 13
- Gems: `dotenv`, `pg`, `httparty`, `cosine-similarity`, `pdf-reader`, `open-uri`, `rack-cors`

## Setup

1. Create and fill in the `.env` file using `env-example` as a blueprint.

2. Install the dependencies.

```bash
bundle install
```

3. Create the database.

```bash
rails db:migrate
```

4. Create the embeddings for the tokens in the CSV file. 

```bash
ruby scripts/csv_embeddings.rb #or  ruby scripts/pdf_embeddings.rb
```

Use the `ruby scripts/csv_embeddings.rb` to create a CSV file with embeddings from the `microsoft-earnings.csv`file. Use the `ruby scripts/pdf_embeddings.rb` to create a CSV file with embeddings from a local pdf file or a pdf file from a URL.

After that, you should be good to Go. Should you choose to deploy it you can remove the `embeddings.csv` file from the `.gitignore` file to deploy the app with the embeddings to save on costs.

## Deploy to Render

1. Create a new app on Render and select the GitHub repo with this application.

2. Create a blueprint using the `render.yaml` file in the root directory of the project. Click [here](https://render.com/docs/deploy-rails#deploy-to-render) for a ruby example. Otherwise, you can deploy manually. Make sure the build commands when deploying manually are `bundle install` and `bundle exec rake db:migrate`, there is no need for `bundle exec rake assets:precompile` since the app doesn't have any assets because it is just an API.

3. Create a postgresql DB on Render and add the DB credentials to the environment variables in the Render dashboard.

4. Generate a secret key for the app using the `rails secret` command and add it to the environment variables in the Render dashboard. Create a `RAILS_MASTER_KEY` and add the secret generated.

You should have a deployed app now.

## Challenges

- Reading Data from the CSV file: When reading data from the CSV file, all the data is read as a string. This posed a problem when trying to calculate the dot product(or cosine similarity) of the embeddings.  I had to use the `JSON.parse()` method to convert the embeddings of a token from a string to an array of floats.

- Working with env variables in production: When deploying to render you don't use the env variables in the .env file. Instead, you use the environment variables in the render dashboard. So the `dotenv` gem doesn't have a `.env`file to read from in production meaning the gem should only be used in development and test environments.


- Calculating Dot Product: In practice, the dot product is similar to the cosine similarity of two vectors. Luckily, I found the cosine-similarity gem which carried out the calculation for me.

## Improvements

- Data collection: I added the `ask_count` column to the `questions` table to keep track of the number of times a question was asked. This will help in the future when I want to add a feature that will help get insight into what people want to know most about the product, this information can be used to improve the product.

- The use of a cache database instead of a file store: I used the file store cache for this project because it was a small project. The file store cache is ideal for low-traffic sites. However, I would like to use a cache database like Redis in the future if this app was to have medium to high traffic.