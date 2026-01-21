# Chapter IV - Background Jobs and Process Management
`(avr. time for this chapter: 1 day)`

This chapter introduces background job processing with Sidekiq and essential process management skills. You will learn how to handle long-running tasks asynchronously and manage system processes effectively.

Continue building upon the application from Chapter III.

## Introduction to Background Jobs

In web applications, some operations are too slow to handle during a regular HTTP request. Examples include sending emails, processing files, generating reports, or calling external APIs. Background jobs allow these operations to run asynchronously, improving user experience and application responsiveness.

### Why Use Background Jobs?

- **Better User Experience** - Users don't wait for slow operations
- **Reliability** - Jobs can be retried if they fail
- **Scalability** - Workers can be scaled independently
- **Scheduling** - Jobs can run at specific times

## Sidekiq Setup

Sidekiq is a popular background job processor for Ruby. It uses Redis to manage job queues and is known for its efficiency and reliability.

### Steps to implement:

1. Add Sidekiq gem to your project
2. Install and start Redis (required by Sidekiq)
3. Configure Active Job to use Sidekiq as the queue adapter
4. Verify the setup by starting Sidekiq and checking for errors

> Reference: [Sidekiq Documentation](https://github.com/sidekiq/sidekiq/wiki/Getting-Started)

## Creating Background Jobs

### Steps to implement:

1. Generate a new job using Rails generator
2. Implement job logic that sends purchase notification emails
3. Call the job from your purchase controller using `perform_later`
4. Experiment with scheduling jobs for future execution

> Note: Always pass primitive types (IDs, strings) to jobs instead of objects. Objects may change between when the job is enqueued and when it runs.

## Practical Exercise: Async Purchase Notifications

Refactor your ebook store to use background jobs for email notifications.

### Steps to implement:

1. Create a `PurchaseNotificationJob` that handles:
   - Sending commission email to the seller
   - Sending statistics email
   - Any other email notifications from Chapter III

2. Update your purchase flow:
   - Keep the database transaction synchronous
   - Move email sending to background jobs (outside the transaction)
   - Ensure the user receives immediate feedback

3. Create a `StatisticsReportJob` that:
   - Calculates daily/weekly ebook statistics
   - Can be scheduled to run periodically

## Running Sidekiq

### Steps to implement:

1. Start Sidekiq in a terminal
2. Configure and mount the Sidekiq Web UI to monitor jobs
3. Make a purchase and observe the job being processed

> Note: In production, Sidekiq should run as a managed process (systemd, Docker, etc.)

## Process Management in Linux/macOS

Understanding how to manage processes is essential for any developer. You will frequently need to start, stop, and debug background processes.

### Steps to implement:

1. Learn to list all running processes using `ps aux`
2. Filter processes by name using `grep`
3. Use `top` or `htop` for interactive process monitoring
4. Understand the key columns in process output:
   - **PID** - Process ID, unique identifier
   - **%CPU** - CPU usage percentage
   - **%MEM** - Memory usage percentage
   - **STAT** - Process state (S=sleeping, R=running, Z=zombie)

## Killing Processes

### Steps to implement:

1. Practice finding process IDs (PIDs) for Sidekiq, Redis, and Ruby processes

2. Learn different ways to terminate processes:
   - Kill a single process by PID
   - Force kill a process that won't terminate
   - Kill all processes matching a name
   - Kill a process using a specific port

3. Understand signal types:
   - **SIGTERM (15)** - Graceful shutdown, process can clean up
   - **SIGKILL (9)** - Immediate termination, no cleanup
   - **SIGINT (2)** - Interrupt (like pressing Ctrl+C)
   - **SIGHUP (1)** - Hangup, often used to reload configuration

> Reference: [Linux Signals](https://man7.org/linux/man-pages/man7/signal.7.html)

### Practical Exercise: Process Management

#### Steps to implement:

1. Start multiple Sidekiq processes with different queue configurations

2. Practice managing these processes:
   - Find all Sidekiq processes and note their PIDs
   - Kill one process gracefully and observe the shutdown behavior
   - Force kill another and compare the behavior
   - Clean up all remaining Sidekiq processes

3. Verify processes are terminated using `ps aux | grep sidekiq`

## Handling Stuck Ports

A common issue when Rails or other servers don't shut down properly.

### Steps to implement:

1. Learn to identify what process is using a specific port
2. Practice killing processes that are blocking ports
3. (Optional) Create a helper function in your shell configuration to simplify this task

## Job Queues and Priorities

### Steps to implement:

1. Configure multiple queues in Sidekiq configuration file (critical, default, low)
2. Assign different jobs to specific queues based on their importance
3. Start Sidekiq with queue priority weights
4. Test that critical jobs are processed before low priority jobs

## Error Handling and Retries

Sidekiq automatically retries failed jobs. Understanding this behavior is crucial.

### Steps to implement:

1. Configure custom retry behavior for specific jobs
2. Implement error handling for:
   - Temporary failures that should be retried
   - Permanent failures that should be discarded
3. Test job failure scenarios and observe retry behavior

## Final Exercise: Complete Integration

### Steps to implement:

1. Refactor all email sending in your ebook store to use background jobs

2. Implement a scheduled job for daily statistics:
   - Create `DailyStatisticsJob`
   - Calculate total sales, popular ebooks, active sellers
   - Discuss scheduling options with your tutor (e.g., `sidekiq-scheduler` gem)

3. Practice the full development workflow:
   - Start Redis, Sidekiq, and Rails server
   - Make a purchase and verify emails are sent asynchronously
   - View job status in Sidekiq Web UI
   - Practice killing and restarting Sidekiq
   - Observe how pending jobs are processed after restart

4. Document the startup process:
   - Create a README section or script that lists all required processes
   - Include commands to start and stop each service

> Tip: Consider using tools like `foreman` or `overmind` to manage multiple processes during development.

## Checklist

Before moving to the next chapter, ensure you can:

- [ ] Explain why background jobs are important
- [ ] Set up and configure Sidekiq with Redis
- [ ] Create and enqueue background jobs
- [ ] Use `ps`, `grep`, `kill`, and `pkill` commands confidently
- [ ] Find and kill processes using specific ports
- [ ] Understand the difference between SIGTERM and SIGKILL
- [ ] Configure job queues and priorities
- [ ] Handle job failures and retries
