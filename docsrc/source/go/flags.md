# Flags


## Ein anderes Beispiel mit Subcommands

https://www.janekbieser.dev/posts/cli-app-with-subcommands-in-go/

```
package main

import (
	"flag"
	"fmt"
	"os"
	"slices"
)

type Command struct {
	Name string
	Help string
	Run  func(args []string) error
}

var commands = []Command{
	{Name: "report", Help: "Create Report", Run: reportCmd},
	{Name: "fetch", Help: "Fetch Data", Run: fetchCmd},
	{Name: "help", Help: "Print this help", Run: printHelpCmd},
}

func printHelpCmd(_ []string) error {
	flag.Usage()
	return nil
}

func reportCmd(args []string) error {
	var (
		reportDate string
		reportCPU  int
		reportNIC  int
		reportGen  string
	)
	reportCmd := flag.NewFlagSet("report", flag.ExitOnError)
	reportCmd.StringVar(&reportDate, "d", "", "Date in format YYYY-MM-DD")
	reportCmd.IntVar(&reportCPU, "c", 0, "Number of vCPUs")
	reportCmd.IntVar(&reportNIC, "n", 0, "Number of network interfaces cards")
	reportCmd.StringVar(&reportGen, "g", "", "HyperV Generation V1 or V2")
	reportCmd.Usage = func() {
		fmt.Fprintln(os.Stderr, `Create APS Report.
			
Usage:
  aps report [flags]
  
  Flags:`)
		reportCmd.PrintDefaults()
		fmt.Fprintln(os.Stderr)
	}
	err := reportCmd.Parse(args)
	if err != nil {
		return fmt.Errorf("error parsing report: %v", err)
	}

	fmt.Println("report", reportCmd.Args())
	fmt.Println(" Date:", reportDate)
	fmt.Println(" CPU:", reportCPU)
	fmt.Println(" NIC:", reportNIC)
	fmt.Println(" Gen:", reportGen)
	return nil
}

func fetchCmd(args []string) error {
	var (
		getData bool
	)
	fetchCmd := flag.NewFlagSet("fetch", flag.ExitOnError)
	fetchCmd.BoolVar(&getData, "f", false, "set to fetch data")
	{
		fmt.Fprintln(os.Stderr, `Fetch Data from Microsoft-
			
Usage:
  asp fetch [flags]
  
  Flags:`)
		fetchCmd.PrintDefaults()
		fmt.Fprintln(os.Stderr)
	}
	err := fetchCmd.Parse(args)
	if err != nil {
		return fmt.Errorf("error parsing report: %v", err)
	}

	fmt.Println("fetch", fetchCmd.Args())
	return nil

}

// func fetchCmd(args []string) error {...}

func customUsage() {

	intro := `Usage: main.go [flags] <command> [command flags]`

	fmt.Fprintf(os.Stderr, "%s", intro)
	fmt.Fprintln(os.Stderr, "\nCommands:")
	for _, cmd := range commands {
		fmt.Fprintf(os.Stderr, " %-8s %s\n", cmd.Name, cmd.Help)
	}

	fmt.Fprintln(os.Stderr, "\nFlags:")
	// Prints a help string for each flag we defined earlier using
	// flag.BoolVar (and related functions)
	flag.PrintDefaults()

	fmt.Fprintln(os.Stderr)
	fmt.Fprintf(os.Stderr, "Run `main.go <command> -h` to get help for a specific command\n\n")

}

func runCommand(name string, args []string) {
	cmdIdx := slices.IndexFunc(commands, func(cmd Command) bool {
		return cmd.Name == name
	})

	if cmdIdx < 0 {
		fmt.Fprintf(os.Stderr, "command \"%s\" not found\n\n", name)
		flag.Usage()
		os.Exit(1)
	}

	if err := commands[cmdIdx].Run(args); err != nil {
		fmt.Fprintf(os.Stderr, "Error: %s", err.Error())
		os.Exit(1)
	}
}

func main() {

	var (
		gloablLogLevel string
	)

	flag.StringVar(&gloablLogLevel, "l", "", "Set loglevel")

	flag.Usage = customUsage
	flag.Parse()

	if len(flag.Args()) < 1 {
		flag.Usage()
		os.Exit(1)
	}

	subCmd := flag.Arg(0)
	subCmdArgs := flag.Args()[1:]

	runCommand(subCmd, subCmdArgs)

}
```