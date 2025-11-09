.PHONY: help run up build attach exec stop down rebuild logs clean

# Default target
.DEFAULT_GOAL := help

# Get project name from parent directory
PROJECT_NAME := $(shell basename $(shell dirname $(CURDIR)))
# Docker Compose v2 names containers as: <project>-<service>-<replica>
CONTAINER_NAME := $(PROJECT_NAME)-dev-1

help: ## Show this help message
	@echo "DevContainer Makefile for: $(PROJECT_NAME)"
	@echo ""
	@echo "Usage: make [target]"
	@echo ""
	@echo "Targets:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "  %-15s %s\n", $$1, $$2}'

run: up attach ## Start and attach to the container (default workflow)

up: ## Start the devcontainer (create if needed)
	@echo "Starting devcontainer: $(CONTAINER_NAME)"
	@if docker ps -a --format '{{.Names}}' | grep -q "^$(CONTAINER_NAME)$$"; then \
		if docker ps --format '{{.Names}}' | grep -q "^$(CONTAINER_NAME)$$"; then \
			echo "Container $(CONTAINER_NAME) is already running"; \
		else \
			echo "Starting existing container..."; \
			docker start $(CONTAINER_NAME); \
		fi \
	else \
		echo "Creating new container..."; \
		COMPOSE_PROJECT_NAME=$(PROJECT_NAME) docker compose up -d; \
	fi

build: ## Build or rebuild the container image
	@echo "Building devcontainer image..."
	COMPOSE_PROJECT_NAME=$(PROJECT_NAME) docker compose build

attach: ## Attach to the running container with zsh
	@if docker ps --format '{{.Names}}' | grep -q "^$(CONTAINER_NAME)$$"; then \
		echo "Attaching to $(CONTAINER_NAME)..."; \
		docker exec -it $(CONTAINER_NAME) zsh; \
	else \
		echo "Error: Container $(CONTAINER_NAME) is not running"; \
		echo "Run 'make up' or 'make run' first"; \
		exit 1; \
	fi

exec: ## Execute a command in the container (e.g., make exec CMD="ls -la")
	@if [ -z "$(CMD)" ]; then \
		echo "Error: CMD not specified"; \
		echo "Usage: make exec CMD=\"your command here\""; \
		exit 1; \
	fi
	@docker exec -it $(CONTAINER_NAME) $(CMD)

stop: ## Stop the container (but keep it)
	@echo "Stopping container: $(CONTAINER_NAME)"
	@docker stop $(CONTAINER_NAME) 2>/dev/null || echo "Container not running"

down: ## Stop and remove the container (keeps volumes)
	@echo "Stopping and removing container: $(CONTAINER_NAME)"
	COMPOSE_PROJECT_NAME=$(PROJECT_NAME) docker compose down

rebuild: ## Rebuild the image and restart the container
	@echo "Rebuilding devcontainer: $(CONTAINER_NAME)"
	COMPOSE_PROJECT_NAME=$(PROJECT_NAME) docker compose down
	COMPOSE_PROJECT_NAME=$(PROJECT_NAME) docker compose build --no-cache
	COMPOSE_PROJECT_NAME=$(PROJECT_NAME) docker compose up -d
	@echo "Rebuild complete. Run 'make attach' to connect."

logs: ## Show container logs
	@docker logs -f $(CONTAINER_NAME)

status: ## Show container status
	@echo "Container status for: $(CONTAINER_NAME)"
	@docker ps -a --filter "name=$(CONTAINER_NAME)" --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" || echo "Container not found"
	@echo ""
	@echo "Volumes:"
	@docker volume ls --filter "name=$(PROJECT_NAME)" --format "table {{.Name}}\t{{.Driver}}" || echo "No volumes found"

clean: ## Remove container and volumes (DESTRUCTIVE!)
	@echo "WARNING: This will remove the container and ALL persistent data (volumes)"
	@printf "Are you sure? [y/N] "; \
	read REPLY; \
	case "$$REPLY" in \
		[Yy]*) \
			echo "Removing container and volumes..."; \
			COMPOSE_PROJECT_NAME=$(PROJECT_NAME) docker compose down -v; \
			echo "Clean complete."; \
			;; \
		*) \
			echo "Cancelled."; \
			;; \
	esac

shell: ## Quick alias for attach
shell: attach

start: ## Alias for up
start: up
