#!/usr/bin/env ruby
# frozen_string_literal: true

require "csv"
require "date"

# Represents a single task
class Task
  attr_accessor :title, :category, :priority, :due_date, :completed

  def initialize(title, category, priority, due_date, completed = false)
    @title = title
    @category = category
    @priority = priority
    @due_date = due_date
    @completed = completed
  end

  def to_a
    [@title, @category, @priority, @due_date, @completed]
  end
end

# Main Task Manager class
class TaskManager
  FILE_PATH = "tasks.csv"

  def initialize
    @tasks = load_tasks
  end

  def add_task
    print "Title: "
    title = gets.chomp

    print "Category: "
    category = gets.chomp

    print "Priority (1-5): "
    priority = gets.chomp.to_i

    print "Due date (YYYY-MM-DD): "
    due_date = gets.chomp

    task = Task.new(title, category, priority, due_date)
    @tasks << task
    save_tasks
    puts "✅ Task added!"
  end

  def list_tasks
    puts "\n📋 Tasks"
    puts "-" * 60
    puts "Title                 | Category        | Priority | Due Date   | Completed"
    puts "-" * 60
    @tasks.each do |task|
      puts "#{task.title.ljust(20)} | #{task.category.ljust(15)} | #{task.priority}       | #{task.due_date} | #{task.completed}"
    end
    puts "-" * 60
  end

  def mark_completed
    list_tasks
    print "Enter task title to mark completed: "
    title = gets.chomp
    task = @tasks.find { |t| t.title.downcase == title.downcase }
    if task
      task.completed = true
      save_tasks
      puts "✅ Task marked as completed!"
    else
      puts "❌ Task not found."
    end
  end

  private

  def load_tasks
    return [] unless File.exist?(FILE_PATH)

    CSV.read(FILE_PATH, headers: true).map do |row|
      Task.new(row["title"], row["category"], row["priority"].to_i, row["due_date"], row["completed"] == "true")
    end
  end

  def save_tasks
    CSV.open(FILE_PATH, "w", write_headers: true, headers: %w[title category priority due_date completed]) do |csv|
      @tasks.each { |task| csv << task.to_a }
    end
  end
end

# -----------------------------------------------------------
# CLI
# -----------------------------------------------------------
manager = TaskManager.new

loop do
  puts "\n📝 Task Manager"
  puts "1. Add Task"
  puts "2. List Tasks"
  puts "3. Mark Task Completed"
  puts "4. Exit"
  print "Select option: "
  choice = gets.chomp.to_i

  case choice
  when 1
    manager.add_task
  when 2
    manager.list_tasks
  when 3
    manager.mark_completed
  when 4
    puts "Goodbye!"
    break
  else
    puts "Invalid choice, try again."
  end
end
