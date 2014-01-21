//see [http://johnmacfarlane.net/pandoc/epub.html](http://johnmacfarlane.net/pandoc/epub.html)

// Create epub/pdf/text from markdown pandoc flavor f sharp script


let writeToFile (fileName:string) obj = 
        printfn "File name: %s" fileName
        use file1 = System.IO.File.CreateText(fileName)
        file1.WriteLine("{0}", obj.ToString())


let readFile (fileName : string) =
    let stream = new System.IO.StreamReader(fileName)
    stream.ReadToEnd()

let pandoc = "pandoc"

let span = sprintf """<span class"%s">%s</span>"""

let regexPDF = [
                  """\*\*(.+?)\*\*""", """\textbf{$1}""" 
                ; """([ |\n])_(.+?)_""", """$1\underline{$2}"""
                ; """\*(.+?)\*""", """\underline{$1}"""
                ; """<[span|p] class="remembrance">(.+?)</[span|p]>""", """\textit{$1}"""
                ; @"<span class=""fauxUnderline"">(.+?)</span>", """\underline{$1}"""
                ; @"#### (.+?)\n", @"#### $1\ " + System.Environment.NewLine + @"\ " + System.Environment.NewLine
                ; @"(&)", @"\$1"
                ; @"\$", @"\$"
                ; """\\&deg;""", @"&deg;"
                ; """\`\`\`(.+?)\`\`\`""", """"\begin{verse}$1\end{verse}"""
                ; @"\^([^\[\]]+?)\^", @"\textsuperscript{$1}"
                ]

let regexEpub = 
    [
        """\`\`\`(.+?)\`\`\`""", (span "poems" "$1")
    ]

let regexText = []

let path = @"C:\Users\Laura\Documents\Jon Nyman KE7WYY\Dropbox\ePubs\Tone-Our_Letters\Text"

let files = [ @"OurLetters.md"; @"Peggy.md" ]

let toEpub = """ -S --epub-metadata=../metadata.xml --epub-stylesheet=../Styles/template.css --epub-cover-image=../Images/0000_FloLetters.JPG -o ../OurLetters_Transcribed.epub """
let toPDF = """ -o ../OurLetters_Transcribed.pdf """
let toText = """ -o ../OurLetters_Transcribed.txt """

let allArguments = [ toEpub; toPDF; toText ]
let allRegex = [ regexEpub; regexPDF; regexText ]

let findAndReplace (text : string) (findReplace : string*string) =
    System.Text.RegularExpressions.Regex.Replace(text, fst findReplace, snd findReplace, System.Text.RegularExpressions.RegexOptions.Singleline)
        
let preProcessFile text regex = List.fold (fun text rgx -> findAndReplace text rgx) text regex

let allFiles (path : string) (files : string list) = 
    files 
    |> List.map (fun file -> readFile (System.IO.Path.Combine(path, file)))

let createNewDocument (path : string) (text : string) (arguments : string) =
    let p = new System.Diagnostics.Process()
    p.StartInfo.FileName <- pandoc
    p.StartInfo.Arguments <- arguments
    p.StartInfo.WindowStyle <- System.Diagnostics.ProcessWindowStyle.Hidden
    p.StartInfo.RedirectStandardError <- true
    p.StartInfo.UseShellExecute <- false
    p.StartInfo.WorkingDirectory <- path
    p.Start() |> ignore
    let error = p.StandardError.ReadToEnd()
    printfn "%s" error
    p.WaitForExit() |> ignore


let morph path texts regex arguments =
    let texts' = texts |> List.map (fun text -> preProcessFile text regex)
    let text = String.concat System.Environment.NewLine texts'
    let temp = (System.IO.Path.Combine(@"C:\Users\Laura\AppData\Local\Temp\", "temp.md")) //System.IO.Path.GetTempPath()
    printfn "%s" temp
    writeToFile temp text
    createNewDocument path text (arguments + " " + temp)

let main =
    let files' = allFiles path files
    List.iter2 (fun arg rgx -> morph path files' rgx arg) allArguments allRegex