# RatatouilleGen - Recipe Generation System

A Python-based AI system that generates novel recipes using RAG (Retrieval-Augmented Generation) with Ollama and Qwen3(predominant) model.

## Important Note on Large Files

Due to GitHub's file size limitations, some large files are not included in the repository. Please download them from the following links and place them in the specified locations:

1. **Pre-built Vector Database**
   - Download: [recipe_chroma_db](https://drive.google.com/drive/folders/1WhSCtmpufp1Q-S6y9i1CfPXSE47rShAV?usp=sharing)
   - Extract the contents to: `./recipe_chroma_db/` directory

2. **Recipe Dataset**
   - Download: [RecipeDB2_general_final.csv](https://drive.google.com/file/d/18s-EP85JeqpLHOBT6Ryb6YZpevM_S6BA/view?usp=sharing)
   - Place in: `./data/` directory

3. **Model Weights** (if not using Ollama)
   - Download: [model_weights.bin](YOUR_DOWNLOAD_LINK_HERE) [ Assume on your own from Hugging Face]
   - Place in: `./models/` directory

## Features

- **AI-Powered Recipe Creation**: Uses Qwen3:8b model via Ollama for intelligent recipe generation
- **Regional Cuisine Adaptation**: Creates novel,authentic recipes adapted to specific regional cooking styles
- **Vector Database Search**: Uses ChromaDB with HuggingFace embeddings for recipe knowledge retrieval(all-miniLM-L6-V2)
- **Dual Interface**: Command-line interface and FastAPI web service
- **Automatic Setup**: Handles Ollama installation and model downloading

## System Requirements

- Python 3.8 or higher
- At least 5GB free disk space
- 8GB+ RAM recommended
- Internet connection for initial setup

## Installation

### 1. Clone/Download the Files (for local development)

Place all the Python files in a directory:
- `main.py` - Command-line interface
- `app.py` - FastAPI web service
- `ollama_manager.py` - Ollama server management
- `vector_db_builder.py` - Vector database builder
- `rag_chain.py` - RAG implementation
- `setup.py` - Setup script
- `requirements.txt` - Python dependencies

### 2. Run Setup (for local)

```bash
python setup.py
```

This will:
- Install all Python dependencies
- Create necessary directories
- Set up HuggingFace token (optional)

### 3. Prepare Your Recipe Data ( for local)

Download the recipe dataset from the link provided in the "Important Note on Large Files" section and place it in the `./data/` directory. The CSV should have these columns:
- `recipe_title`
- `description`
- `region`
- `subregion`
- `ingredients`
- `servings`
- `prep_time`
- `cook_time`
- `steps`
- `author` (optional)
- `vegan` (optional)

## Usage

### Approach 1: Local Development with LLM (Slower but Simple)

This approach runs everything locally, including the language model. It's simpler to set up but will be slower due to running the LLM on your local machine.

#### Prerequisites
- Ensure you have enough RAM (16GB+ recommended)
- Ollama installed and running
- Sufficient disk space for model weights

#### Steps:

1. **Prepare Vector Database** 
   - If using pre-built database: Extract the downloaded `recipe_chroma_db.zip` to the project root
   - Or build your own:
     ```bash
     python vector_db_builder.py --csv ./data/your_recipe_file.csv
     ```

2. **Run in Interactive Mode**
   ```bash
   python main.py --csv ./data/your_recipe_file.csv
   ```
   - Follow the prompts to generate recipes interactively

3. **Run Single Query**
   ```bash
   python main.py --csv ./data/your_recipe_file.csv --query "Italian: tomatoes, basil, mozzarella, pasta"
   ```

#### Advantages:
- No internet required after initial setup
- Complete data privacy
- Simpler debugging

#### Disadvantages:
- Slower response times
- Higher system resource usage
- Limited by local hardware

### Approach 2: FastAPI Web Service (Recommended for Production)

This approach runs the model through a web service, which can be deployed on more powerful hardware.

#### Prerequisites
- Python 3.8+
- Required Python packages (install via `pip install -r requirements.txt`)
- Vector database (either pre-built or custom-built)

#### Steps:

1. **Prepare Vector Database**( already prepared, place it in the correct folder directory)
   - Extract the pre-built database to `./recipe_chroma_db/`
   - Or build your own:
     ```bash
     python vector_db_builder.py --csv ./data/your_recipe_file.csv
     ```

2. **Start the FastAPI Server**
   ```bash
   python app.py
   ```
   - Server starts on `http://localhost:8000`
   - API docs available at `http://localhost:8000/docs`

3. **Make API Requests**
   ```bash
   curl -X POST "http://localhost:8000/generate-recipe" \
        -H "Content-Type: application/json" \
        -d '{"region":"Italian", "ingredients":"tomatoes, basil, mozzarella, pasta"}'
   ```

#### Advantages:
- Better performance with proper hardware
- Can be scaled horizontally
- Can be deployed to cloud services
- Standardized API interface

#### Disadvantages:
- Requires more initial setup
- Needs to be properly secured for production use

The server will start on `http://localhost:8000` with:
- API documentation at `/docs`
- ReDoc documentation at `/redoc`
- Health check at `/health-check`

### API Endpoints

- `POST /generate-recipe` - Generate a new recipe
  - Request body:
    ```json
    {
      "region": "string",
      "ingredients": "string"
    }
    ```
  - Response:
    ```json
    {
      "recipe": "string",
      "sources": [array of source documents],
      "status": "string"
    }
    ```

- `GET /health-check` - Check server status

### Query Format

Enter queries in one of these formats:
- `Region: ingredients` - e.g., "Italian: tomatoes, basil, pasta"
- `ingredients only` - defaults to Indian cuisine

Examples:
- `South American: Grapeseed oil, Hash Browns, Carrots, Green onions, Goat cheese, Sun-dried tomato, Eggs`
- `Japanese: Rice, Nori, Salmon, Cucumber, Avocado`
- `Chicken, Rice, Garam masala` (defaults to Indian)

## File Structure

```
recipe-system/
├── app.py                 # FastAPI web service
├── main.py               # Command-line interface
├── ollama_manager.py     # Ollama server management
├── vector_db_builder.py  # Builds vector database from CSV
├── rag_chain.py         # RAG implementation
├── setup.py             # Setup script
├── requirements.txt     # Python dependencies
├── README.md           # This file
├── data/               # Place CSV files here
├── recipe_chroma_db/   # Vector database (auto-created)
└── models/            # Model cache (auto-created)
```

## Command Line Options

### For main.py:
- `--csv PATH`: Path to recipe CSV file
- `--model NAME`: Ollama model name (default: qwen2:7b)
- `--rebuild-db`: Force rebuild of vector database
- `--query TEXT`: Process single query instead of interactive mode

### For app.py:
- No command line arguments needed, runs as a web service
- Configuration is handled through environment variables and code settings

## Troubleshooting

### Ollama Installation Issues

If Ollama fails to install automatically:

**Linux/macOS:**
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

**Windows:**
Download from: https://ollama.com/download

### Model Download Issues

Manually pull the model:
```bash
ollama pull qwen3:8b // or whatever model from ollama you want to pull
```

### Vector Database Issues

Force rebuild the database: (optional if not retraining for RAG)
```bash
python main.py --csv ./data/your_file.csv --rebuild-db
```

### Memory Issues

- Use a smaller model: `--model qwen2:1.5b`
- Reduce chunk size in `vector_db_builder.py`
- Close other applications

## Dependencies

See `requirements.txt` for all dependencies. Key components:
- `langchain` - RAG framework
- `chromadb` - Vector database
- `ollama` - AI model serving
- `sentence-transformers` - Text embeddings
- `pandas` - Data processing

## How It Works

1. **Vector Database**: Recipes from CSV are embedded and stored in ChromaDB
2. **Query Processing**: User queries are parsed for region and ingredients
3. **Retrieval**: Similar recipes are found using vector similarity search
4. **Generation**: Qwen2 model creates novel recipes using retrieved context
5. **Output**: Formatted recipes with regional authenticity and cooking instructions

## Output Format

Generated recipes include:
- **Recipe Title**: 3-word summary reflecting regional style
- **Taste Profile**: Expected flavor characteristics
- **Regional Context**: Cultural authenticity explanation
- **Ingredient List**: Precise measurements for 4 servings
- **Preparation Steps**: Detailed cooking instructions
- **Chef's Notes**: Cultural insights and technique explanations

## Contributing

Feel free to modify and extend the system:
- Add new embedding models in `vector_db_builder.py`
- Customize prompts in `rag_chain.py`
- Add new output formats in `main.py`
- Enhance Ollama management in `ollama_manager.py`

## License
Under CoSy Lab headed by Prof. Dr. Ganesh Bagler