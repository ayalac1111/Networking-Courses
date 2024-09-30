# Courses Automation Repository

This repository contains automation scripts, configurations, and lab instructions for courses taught at Algonquin College. Each course is organized by term and course number in the format `TERM-COURSE_NUMBER` (e.g., `24F-NET3008`).

## Directory Structure

Each course will have its directory, named as follows:

- **`TERM-COURSE_NUMBER/`**
  - **Labs/**: Contains lab configurations, scripts, and any relevant files for hands-on lab activities.
  - **Scripts/**: Contains automation scripts related to network configurations or other course-related tasks.
  - **Configs/**: Stores configuration files that are used in the labs, including router/switch configurations or system configurations.
  - **README.md**: Course-specific README files providing details about lab objectives, requirements, and setup instructions.

### Example:
``` bash
Courses-Automation/
├── 24F-NET3008
│   └── ISIS-Lab
│       └── isis-lab-commands.yaml
└── README.md
```

## Usage

To use the materials in this repository:

1. Navigate to the relevant course directory.
2. Review the `README.md` file for that course to understand the lab structure and requirements.
3. Execute the scripts or follow the lab instructions as outlined.

## Contributing

If you are a student or contributor, please follow these steps:

1. Fork the repository.
2. Create a new branch (`git checkout -b my-feature`).
3. Commit your changes (`git commit -am 'Add some feature'`).
4. Push to the branch (`git push origin my-feature`).
5. Open a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
