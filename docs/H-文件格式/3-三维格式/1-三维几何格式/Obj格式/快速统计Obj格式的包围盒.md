测试数据：139Mb

## 方法一：6186ms
```cpp
BoundingBox box;

std::ifstream in(inFile);
if (!in)
{
    return {};
}

for (std::string tmp; getline(in, tmp); )
{
    std::istringstream lineIn(tmp);

    std::string flag;
    lineIn >> flag;

    if (flag == "v")
    {
        double xyz[3];
        for (int i = 0; i < 3 && !lineIn.eof(); ++i)
        {
            lineIn >> xyz[i];
        }

        box.expandBy(xyz[0], xyz[1], xyz[2]);
    }
}
```

## 方法二：2980ms

```cpp
BoundingBox calObjAABB(const std::string& inFile)
{
    std::ifstream in(inFile);
    if (!in)
    {
        std::cerr << "[Warning] " << inFile << "Open Failed!" << std::endl;
        return {};
    }

    auto safeGetline = [](std::istream &is, std::string &t)->std::istream&
    {
        t.clear();

        std::istream::sentry se(is, true);
        std::streambuf *sb = is.rdbuf();

        if (se) {
            for (;;) {
                int c = sb->sbumpc();
                switch (c) {
                case '\n':
                    return is;
                case '\r':
                    if (sb->sgetc() == '\n') sb->sbumpc();
                    return is;
                case EOF:
                    // Also handle the case when the last line has no line ending
                    if (t.empty()) is.setstate(std::ios::eofbit);
                    return is;
                default:
                    t += static_cast<char>(c);
                }
            }
        }

        return is;
    };
    auto isSpace = [](const char& x)
    {
        return (((x) == ' ') || ((x) == '\t'));
    };
    auto isDigit = [](const char& x)->bool
    {
        return (static_cast<unsigned int>((x)-'0') < static_cast<unsigned int>(10));
    };
    auto tryParseDouble = [&isDigit](const char *s, const char *s_end, double *result)
    {
        if (s >= s_end) 
        {
            return false;
        }

        double mantissa = 0.0;
        // This exponent is base 2 rather than 10.
        // However the exponent we parse is supposed to be one of ten,
        // thus we must take care to convert the exponent/and or the
        // mantissa to a * 2^E, where a is the mantissa and E is the
        // exponent.
        // To get the final double we will use ldexp, it requires the
        // exponent to be in base 2.
        int exponent = 0;

        // NOTE: THESE MUST BE DECLARED HERE SINCE WE ARE NOT ALLOWED
        // TO JUMP OVER DEFINITIONS.
        char sign = '+';
        char exp_sign = '+';
        char const *curr = s;

        // How many characters were read in a loop.
        int read = 0;
        // Tells whether a loop terminated due to reaching s_end.
        bool end_not_reached = false;
        bool leading_decimal_dots = false;

        /*
                BEGIN PARSING.
        */

        // Find out what sign we've got.
        if (*curr == '+' || *curr == '-') {
            sign = *curr;
            curr++;
            if ((curr != s_end) && (*curr == '.')) {
                // accept. Somethig like `.7e+2`, `-.5234`
                leading_decimal_dots = true;
            }
        }
        else if (isDigit(*curr)) { /* Pass through. */
        }
        else if (*curr == '.') {
            // accept. Somethig like `.7e+2`, `-.5234`
            leading_decimal_dots = true;
        }
        else {
            goto fail;
        }

        // Read the integer part.
        end_not_reached = (curr != s_end);
        if (!leading_decimal_dots) {
            while (end_not_reached && isDigit(*curr)) {
                mantissa *= 10;
                mantissa += static_cast<int>(*curr - 0x30);
                curr++;
                read++;
                end_not_reached = (curr != s_end);
            }

            // We must make sure we actually got something.
            if (read == 0) goto fail;
        }

        // We allow numbers of form "#", "###" etc.
        if (!end_not_reached) goto assemble;

        // Read the decimal part.
        if (*curr == '.') {
            curr++;
            read = 1;
            end_not_reached = (curr != s_end);
            while (end_not_reached && isDigit(*curr)) {
                static const double pow_lut[] = {
                    1.0, 0.1, 0.01, 0.001, 0.0001, 0.00001, 0.000001, 0.0000001,
                };
                const int lut_entries = sizeof pow_lut / sizeof pow_lut[0];

                // NOTE: Don't use powf here, it will absolutely murder precision.
                mantissa += static_cast<int>(*curr - 0x30) *
                    (read < lut_entries ? pow_lut[read] : std::pow(10.0, -read));
                read++;
                curr++;
                end_not_reached = (curr != s_end);
            }
        }
        else if (*curr == 'e' || *curr == 'E') {
        }
        else {
            goto assemble;
        }

        if (!end_not_reached) goto assemble;

        // Read the exponent part.
        if (*curr == 'e' || *curr == 'E') {
            curr++;
            // Figure out if a sign is present and if it is.
            end_not_reached = (curr != s_end);
            if (end_not_reached && (*curr == '+' || *curr == '-')) {
                exp_sign = *curr;
                curr++;
            }
            else if (isDigit(*curr)) { /* Pass through. */
            }
            else {
                // Empty E is not allowed.
                goto fail;
            }

            read = 0;
            end_not_reached = (curr != s_end);
            while (end_not_reached && isDigit(*curr)) {
                if (exponent > std::numeric_limits<int>::max() / 10) {
                    // Integer overflow
                    goto fail;
                }
                exponent *= 10;
                exponent += static_cast<int>(*curr - 0x30);
                curr++;
                read++;
                end_not_reached = (curr != s_end);
            }
            exponent *= (exp_sign == '+' ? 1 : -1);
            if (read == 0) goto fail;
        }

    assemble:
        *result = (sign == '+' ? 1 : -1) *
            (exponent ? std::ldexp(mantissa * std::pow(5.0, exponent), exponent)
                : mantissa);
        return true;
    fail:
        return false;
    };
    auto parseReal = [&tryParseDouble](const char **token, double default_value = 0.0)->double
    {
        (*token) += strspn((*token), " \t");
        const char *end = (*token) + strcspn((*token), " \t\r");
        double val = default_value;
        tryParseDouble((*token), end, &val);
        double f = static_cast<double>(val);
        (*token) = end;
        return f;
    };

    BoundingBox box;

    std::string linebuf;
    while (in.peek() != -1)
    {
        safeGetline(in, linebuf);

        // Trim newline '\r\n' or '\n'
        if (linebuf.size() > 0)
        {
            if (linebuf[linebuf.size() - 1] == '\n')
                linebuf.erase(linebuf.size() - 1);
        }
        if (linebuf.size() > 0)
        {
            if (linebuf[linebuf.size() - 1] == '\r')
                linebuf.erase(linebuf.size() - 1);
        }

        // Skip if empty line.
        if (linebuf.empty())
        {
            continue;
        }

        // Skip leading space.
        const char *token = linebuf.c_str();
        token += strspn(token, " \t");

        if (token[0] == 'v' && isSpace(token[1]))
        {
            token += 2;

            double x = parseReal(&token);
            double y = parseReal(&token);
            double z = parseReal(&token);

            box.expandBy(x, y, z);
        }
    }

    return box;
}
```