m_fullFilenameWithPath was reaplcead with getFilename, does this call have a perf impact?
what if virtual?

std::string HtmlTextConverter::convertToHtml() 
{
    std::ifstream reader(getFilename());

    std::string line;
    std::string html;
    while (std::getline(reader,line))
    {
        html += StringEscapeUtils::escapeHtml(line);
        html += "<br />";
    }
    return html;
}

std::string HtmlTextConverter::getFilename() 
{
    return m_fullFilenameWithPath;
}
