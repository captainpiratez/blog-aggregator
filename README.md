# Gator

A command-line RSS feed aggregator built in Go. Gator allows you to follow multiple RSS feeds, aggregate posts, and browse them all in one place.

## Prerequisites

Before you can run Gator, you'll need to have the following installed on your system:

1. **Go** (version 1.25 or higher)
   - Download and install from [golang.org](https://golang.org/dl/)
   - Verify installation: `go version`

2. **PostgreSQL** (version 12 or higher)
   - Download and install from [postgresql.org](https://www.postgresql.org/download/)
   - Make sure PostgreSQL is running on your system
   - Create a database for the application (e.g., `gator`)
   - Verify installation: `psql --version`
   
   **Alternative: Using Docker**
   
   If you prefer using Docker, you can run PostgreSQL in a container:
   ```bash
   docker run --name gator-postgres -e POSTGRES_PASSWORD=yourpassword -e POSTGRES_DB=gator -p 5432:5432 -d postgres:latest
   ```
   
   This will create and start a PostgreSQL container with:
   - Container name: `gator-postgres`
   - Database: `gator`
   - Password: `yourpassword` (change this!)
   - Port: 5432 (mapped to your host)
   
   To stop the container: `docker stop gator-postgres`
   
   To start it again: `docker start gator-postgres`

## Installation

Install the Gator CLI using Go's built-in install command:

```bash
go install github.com/captainpiratez/gator@latest
```

This will compile the program and place the `gator` binary in your `$GOPATH/bin` directory (usually `~/go/bin`). Make sure this directory is in your system's PATH.

To add it to your PATH (if not already added), add this line to your `~/.bashrc` or `~/.zshrc`:

```bash
export PATH=$PATH:$(go env GOPATH)/bin
```

## Configuration

Gator requires a configuration file to connect to your PostgreSQL database.

1. Create a file named `.gatorconfig.json` in your home directory:

```bash
touch ~/.gatorconfig.json
```

2. Add the following content to the file:

```json
{
  "db_url": "postgres://username:password@localhost:5432/gator?sslmode=disable",
  "current_user_name": ""
}
```

Replace `username`, `password`, and `gator` with your PostgreSQL credentials and database name.

**Note:** The `current_user_name` field will be automatically populated when you register or login.

## Database Setup

Before running the application, you need to set up your database schema using [goose](https://github.com/pressly/goose), a database migration tool.

1. Install goose:
   ```bash
   go install github.com/pressly/goose/v3/cmd/goose@latest
   ```

2. Run the migrations to create the necessary tables:
   ```bash
   cd sql/schema
   goose postgres "postgres://username:password@localhost:5432/gator?sslmode=disable" up
   ```
   
   Replace `username`, `password`, and `gator` with your PostgreSQL credentials and database name (should match your `.gatorconfig.json`).

That's it! Goose will automatically run all the migration files in order and set up your database schema.

### Useful Goose Commands

- **Check migration status:**
  ```bash
  goose postgres "your_connection_string" status
  ```

- **Rollback the last migration:**
  ```bash
  goose postgres "your_connection_string" down
  ```

- **Reset database (rollback all migrations):**
  ```bash
  goose postgres "your_connection_string" reset
  ```

## Usage

After installation and configuration, you can run Gator from anywhere in your terminal. The basic syntax is:

```bash
gator <command> [arguments]
```

### Available Commands

#### User Management

- **register** - Create a new user account
  ```bash
  gator register <username>
  ```
  Example: `gator register john`

- **login** - Login as an existing user
  ```bash
  gator login <username>
  ```
  Example: `gator login john`

- **users** - List all registered users
  ```bash
  gator users
  ```

- **reset** - Delete all users from the database (use with caution!)
  ```bash
  gator reset
  ```

#### Feed Management

- **addfeed** - Add a new RSS feed to follow (requires login)
  ```bash
  gator addfeed <feed_name> <feed_url>
  ```
  Example: `gator addfeed "Boot.dev Blog" https://blog.boot.dev/index.xml`

- **feeds** - List all feeds in the database
  ```bash
  gator feeds
  ```

#### Following Feeds

- **follow** - Follow an existing feed (requires login)
  ```bash
  gator follow <feed_url>
  ```
  Example: `gator follow https://blog.boot.dev/index.xml`

- **following** - List all feeds you're currently following (requires login)
  ```bash
  gator following
  ```

- **unfollow** - Unfollow a feed (requires login)
  ```bash
  gator unfollow <feed_url>
  ```

#### Browsing Posts

- **agg** - Aggregate posts from all feeds
  ```bash
  gator agg <time_between_requests>
  ```
  Example: `gator agg 1m` (fetches feeds every 1 minute)
  
  Time format examples: `30s` (30 seconds), `1m` (1 minute), `5m` (5 minutes)

- **browse** - Browse posts from feeds you follow (requires login)
  ```bash
  gator browse [limit]
  ```
  Example: `gator browse 10` (shows the 10 most recent posts)
  
  Default limit: 2 posts

## Example Workflow

Here's a typical workflow to get started with Gator:

```bash
# 1. Register a new user
gator register alice

# 2. Add some feeds
gator addfeed "Boot.dev Blog" https://blog.boot.dev/index.xml
gator addfeed "Go Blog" https://go.dev/blog/feed.atom

# 3. Follow the feeds you added
gator follow https://blog.boot.dev/index.xml
gator follow https://go.dev/blog/feed.atom

# 4. Start the aggregator in the background (fetches new posts every minute)
gator agg 1m

# 5. Browse the latest posts
gator browse 5
```

## Development

If you want to contribute or modify the code:

1. Clone the repository:
   ```bash
   git clone https://github.com/captainpiratez/gator.git
   cd gator
   ```

2. Install dependencies:
   ```bash
   go mod download
   ```

3. Run in development mode:
   ```bash
   go run .
   ```

4. Build the binary:
   ```bash
   go build -o gator
   ```

## Architecture

- **Commands**: User-facing CLI commands are registered and handled in `main.go` and `commands.go`
- **Handlers**: Business logic for each command is in `handler_*.go` files
- **Database**: SQL queries are managed using [sqlc](https://sqlc.dev/) in the `internal/database` package
- **Config**: User configuration is managed in `internal/config/config.go`
- **RSS Parsing**: Feed fetching and parsing logic is in `rss_feed.go`

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
